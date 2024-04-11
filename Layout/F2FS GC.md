# Garbage Collection

```c
gc_thread_func()

do{
	// Mount option으로 GC_MERGE가 켜져있고, foreground gc wq에 수행중인 work queue가 남아있으면, Foreground gc 수행
	if (test_opt(sbi, GC_MERGE) && waitqueue_active(fggc_wq))
		foreground = true;

	
}while


static int gc_thread_func(void *data)
{
	struct f2fs_sb_info *sbi = data;
	struct f2fs_gc_kthread *gc_th = sbi->gc_thread;
	wait_queue_head_t *wq = &sbi->gc_thread->gc_wait_queue_head;
	wait_queue_head_t *fggc_wq = &sbi->gc_thread->fggc_wq;
	unsigned int wait_ms;
	struct f2fs_gc_control gc_control = {
		.victim_segno = NULL_SEGNO,
		.should_migrate_blocks = false,
		.err_gc_skipped = false };

	wait_ms = gc_th->min_sleep_time;

	set_freezable();
//kthread stop요청이 있을 때 까지 반복
	do {
		bool sync_mode, foreground = false;

		wait_event_interruptible_timeout(*wq,
				kthread_should_stop() || freezing(current) ||
				waitqueue_active(fggc_wq) ||
				gc_th->gc_wake,
				msecs_to_jiffies(wait_ms));

// Mount option으로 GC_MERGE가 켜져있고, foreground gc wq에 수행중인 work queue가 남아있으면, Foreground gc 수행
		if (test_opt(sbi, GC_MERGE) && waitqueue_active(fggc_wq))
			foreground = true;

		/* give it a try one time */
		if (gc_th->gc_wake)
			gc_th->gc_wake = false;

// Thread가 freeze되고, sb가 readonly라면, (sbi)->other_skip_bggc++
		if (try_to_freeze() || f2fs_readonly(sbi->sb)) {
			stat_other_skip_bggc_count(sbi);
			continue;
		}

//kthread stop요청이 있었으면, break
		if (kthread_should_stop())
			break;

// Freeze 상태가 write freeze 이상 (SB_FREEZE_WRITE / SB_FREEZE_PAGEFAULT / SB_FREEZE_FS / SB_FREEZE_COMPLETE) 이면 skip
		if (sbi->sb->s_writers.frozen >= SB_FREEZE_WRITE) {
			increase_sleep_time(gc_th, &wait_ms);
			stat_other_skip_bggc_count(sbi);
			continue;
		}

		if (time_to_inject(sbi, FAULT_CHECKPOINT))
			f2fs_stop_checkpoint(sbi, false,
					STOP_CP_REASON_FAULT_INJECT);

// percpu_down_read_trylock(sb->s_writers.rw_sem + level - 1) 이 실패하면, skip
// level == SB_FREEZE_WRITE == 1
		if (!sb_start_write_trylock(sbi->sb)) {
			stat_other_skip_bggc_count(sbi);
			continue;
		}

		/*
		 * [GC triggering condition]
		 * 0. GC is not conducted currently.
		 * 1. There are enough dirty segments.
		 * 2. IO subsystem is idle by checking the # of writeback pages.
		 * 3. IO subsystem is idle by checking the # of requests in
		 *    bdev's request list.
		 *
		 * Note) We have to avoid triggering GCs frequently.
		 * Because it is possible that some segments can be
		 * invalidated soon after by user update or deletion.
		 * So, I'd like to wait some time to collect dirty segments.
		 */
// Urgent 상태이면, wait_ms = 500ms로 하고 f2fs_down_write(&sbi->gc_lock)
		if (sbi->gc_mode == GC_URGENT_HIGH ||
				sbi->gc_mode == GC_URGENT_MID) {
			wait_ms = gc_th->urgent_sleep_time;
			f2fs_down_write(&sbi->gc_lock);
			goto do_gc;
		}

// foreground로 수행하면, f2fs_down_write(&sbi->gc_lock)
		if (foreground) {
			f2fs_down_write(&sbi->gc_lock);
			goto do_gc;
// background 로 수행하면, f2fs_down_write_trylock(&sbi->gc_lock), lock 실패 시 skip
		} else if (!f2fs_down_write_trylock(&sbi->gc_lock)) {
			stat_other_skip_bggc_count(sbi);
			goto next;
		}

// GC Time (def = 5초) 만큼 idle 시간이 지나지 않았으면, lock 해재한 뒤 skip
		if (!is_idle(sbi, GC_TIME)) {
			increase_sleep_time(gc_th, &wait_ms);
			f2fs_up_write(&sbi->gc_lock);
			stat_io_skip_bggc_count(sbi);
			goto next;
		}

// User Block 중 Invalid block counts 40% 초과 && Free block이 Invalid block counts의 40% 미만 일 때, Sleep time 30초 (Min)
		if (has_enough_invalid_blocks(sbi))
			decrease_sleep_time(gc_th, &wait_ms);
// 아닐 때 60 초 (Max)
		else
			increase_sleep_time(gc_th, &wait_ms);
do_gc:
// background gc면, bggc count 증가
		if (!foreground)
			stat_inc_bggc_count(sbi->stat_info);

// Mount option 중 BGGC_MODE_SYNC 가 설정되어있으면 sync_mode = true
		sync_mode = F2FS_OPTION(sbi).bggc_mode == BGGC_MODE_SYNC;

		/* foreground GC was been triggered via f2fs_balance_fs() */
// 그럼에도 foregrounc gc면, sync_mode = false
?? (f2fs_balance_fs() 는 sync_mode gc랑 무슨 연관?)
		if (foreground)
			sync_mode = false;

// sync_mode = true (FG), false (BG)
		gc_control.init_gc_type = sync_mode ? FG_GC : BG_GC;
		gc_control.no_bg_gc = foreground;
		gc_control.nr_free_secs = foreground ? 1 : 0;

// f2fs_gc 수행
// background gc에서 victim 이 없는 경우라면, wait_ms = 300초
		/* if return value is not zero, no victim was selected */
		if (f2fs_gc(sbi, &gc_control)) {
			/* don't bother wait_ms by foreground gc */
			if (!foreground)
				wait_ms = gc_th->no_gc_sleep_time;
		} else {
			/* reset wait_ms to default sleep time */
			if (wait_ms == gc_th->no_gc_sleep_time)
				wait_ms = gc_th->min_sleep_time;
		}

// foregrouond gc라면, fggc work queue 모두 깨움
		if (foreground)
			wake_up_all(&gc_th->fggc_wq);

		trace_f2fs_background_gc(sbi->sb, wait_ms,
				prefree_segments(sbi), free_segments(sbi));

		/* balancing f2fs's metadata periodically */
		f2fs_balance_fs_bg(sbi, true);
next:
// gc_mode가 GC_NORMAL이 아니면, (GC_IDLE, GC_URGENT) remain trials count 감소, 
// remaining_trials 가 0이되면 (모두감소) gc_mode = GC_NORMAL
		if (sbi->gc_mode != GC_NORMAL) {
			spin_lock(&sbi->gc_remaining_trials_lock);
			if (sbi->gc_remaining_trials) {
				sbi->gc_remaining_trials--;
				if (!sbi->gc_remaining_trials)
					sbi->gc_mode = GC_NORMAL;
			}
			spin_unlock(&sbi->gc_remaining_trials_lock);
		}
		sb_end_write(sbi->sb);

	} while (!kthread_should_stop());
	return 0;
}
```

```c
int f2fs_gc(struct f2fs_sb_info *sbi, struct f2fs_gc_control *gc_control)
{
	int gc_type = gc_control->init_gc_type;
	unsigned int segno = gc_control->victim_segno;
	int sec_freed = 0, seg_freed = 0, total_freed = 0, total_sec_freed = 0;
	int ret = 0;
	struct cp_control cpc;
	struct gc_inode_list gc_list = {
		.ilist = LIST_HEAD_INIT(gc_list.ilist),
		.iroot = RADIX_TREE_INIT(gc_list.iroot, GFP_NOFS),
	};
	unsigned int skipped_round = 0, round = 0;
	unsigned int upper_secs;

	trace_f2fs_gc_begin(sbi->sb, gc_type, gc_control->no_bg_gc,
				gc_control->nr_free_secs,
				get_pages(sbi, F2FS_DIRTY_NODES),
				get_pages(sbi, F2FS_DIRTY_DENTS),
				get_pages(sbi, F2FS_DIRTY_IMETA),
				free_sections(sbi),
				free_segments(sbi),
				reserved_segments(sbi),
				prefree_segments(sbi));

	cpc.reason = __get_cp_reason(sbi);
gc_more:
	sbi->skipped_gc_rwsem = 0;
	if (unlikely(!(sbi->sb->s_flags & SB_ACTIVE))) {
		ret = -EINVAL;
		goto stop;
	}
	if (unlikely(f2fs_cp_error(sbi))) {
		ret = -EIO;
		goto stop;
	}

	/* Let's run FG_GC, if we don't have enough space. */
	if (has_not_enough_free_secs(sbi, 0, 0)) {
		gc_type = FG_GC;

		/*
		 * For example, if there are many prefree_segments below given
		 * threshold, we can make them free by checkpoint. Then, we
		 * secure free segments which doesn't need fggc any more.
		 */
		if (prefree_segments(sbi)) {
			ret = f2fs_write_checkpoint(sbi, &cpc);
			if (ret)
				goto stop;
			/* Reset due to checkpoint */
			sec_freed = 0;
		}
	}

	/* f2fs_balance_fs doesn't need to do BG_GC in critical path. */
	if (gc_type == BG_GC && gc_control->no_bg_gc) {
		ret = -EINVAL;
		goto stop;
	}
retry:
	ret = __get_victim(sbi, &segno, gc_type);
	if (ret) {
		/* allow to search victim from sections has pinned data */
		if (ret == -ENODATA && gc_type == FG_GC &&
				f2fs_pinned_section_exists(DIRTY_I(sbi))) {
			f2fs_unpin_all_sections(sbi, false);
			goto retry;
		}
		goto stop;
	}

	seg_freed = do_garbage_collect(sbi, segno, &gc_list, gc_type,
				gc_control->should_migrate_blocks);
	total_freed += seg_freed;

	if (seg_freed == f2fs_usable_segs_in_sec(sbi, segno)) {
		sec_freed++;
		total_sec_freed++;
	}

	if (gc_type == FG_GC) {
		sbi->cur_victim_sec = NULL_SEGNO;

		if (has_enough_free_secs(sbi, sec_freed, 0)) {
			if (!gc_control->no_bg_gc &&
			    total_sec_freed < gc_control->nr_free_secs)
				goto go_gc_more;
			goto stop;
		}
		if (sbi->skipped_gc_rwsem)
			skipped_round++;
		round++;
		if (skipped_round > MAX_SKIP_GC_COUNT &&
				skipped_round * 2 >= round) {
			ret = f2fs_write_checkpoint(sbi, &cpc);
			goto stop;
		}
	} else if (has_enough_free_secs(sbi, 0, 0)) {
		goto stop;
	}

	__get_secs_required(sbi, NULL, &upper_secs, NULL);

	/*
	 * Write checkpoint to reclaim prefree segments.
	 * We need more three extra sections for writer's data/node/dentry.
	 */
	if (free_sections(sbi) <= upper_secs + NR_GC_CHECKPOINT_SECS &&
				prefree_segments(sbi)) {
		ret = f2fs_write_checkpoint(sbi, &cpc);
		if (ret)
			goto stop;
		/* Reset due to checkpoint */
		sec_freed = 0;
	}
go_gc_more:
	segno = NULL_SEGNO;
	goto gc_more;

stop:
	SIT_I(sbi)->last_victim[ALLOC_NEXT] = 0;
	SIT_I(sbi)->last_victim[FLUSH_DEVICE] = gc_control->victim_segno;

	if (gc_type == FG_GC)
		f2fs_unpin_all_sections(sbi, true);

	trace_f2fs_gc_end(sbi->sb, ret, total_freed, total_sec_freed,
				get_pages(sbi, F2FS_DIRTY_NODES),
				get_pages(sbi, F2FS_DIRTY_DENTS),
				get_pages(sbi, F2FS_DIRTY_IMETA),
				free_sections(sbi),
				free_segments(sbi),
				reserved_segments(sbi),
				prefree_segments(sbi));

	f2fs_up_write(&sbi->gc_lock);

	put_gc_inode(&gc_list);

	if (gc_control->err_gc_skipped && !ret)
		ret = total_sec_freed ? 0 : -EAGAIN;
	return ret;
}
```


GC 가 빈번하게 요구되는 상황에서는 balance fs 에서 foreground로 호출되고 (일반적으로 1개 free segment 획득이 목적),
일반 적으로는 background gc 로 동작.



gc_thread_func()
	f2fs_down_write(&sbi->gc_lock)
	-> f2fs_gc()
GC_MORE:
		if has_not_enough_free_secs()
			gc_type = FG_GC
			if prefree_segment > 0
				f2fs_write_checkpoint()
RETRY:
		__get_victim() - <need to search>
			down_write(&sit_i->sentry_lock)
			f2fs_get_victim()
			up_write(&sit_i->sentry_lock)
		do_garbage_collect() - <need to search>
		f2fs_usable_segs_in_sec()
		if FG_GC
			if has_not_enough_free_secs()
				goto STOP
		else if has_not_enough_free_secs()
			goto STOP
		__get_secs_required()
		goto GC_MORE
STOP:
	f2fs_up_write(&sbi->gc_lock)




gc_data_segment()에서 sbi->skipped_gc_rwsem++ 발생, phase 의미는?


do_garbage_collect()
	f2fs_get_sum_page() - 여기서 Summary page 가져오고, 아래서 또 가져오는 이유는?
		__get_meta_page()
			page = f2fs_grab_cache_page()
				grab_cache_page()
  					__filemap_get_folio() - folio_lock(folio)
			f2fs_submit_page_bio()
			lock_page(page)

	sum_page = find_get_page()
	(case 1)gc_node_segment()
		f2fs_ra_meta_pages() : Phase 1, nid에 해당하는 NAT Entry read (for block addr of NODE)
		f2fs_ra_node_page() : Phase 2
		f2fs_get_node_page() : Phase 3
		f2fs_get_node_info()
		f2fs_move_node_page()
	(case 2)gc_data_segment()
		f2fs_ra_meta_pages() : Phase 1
		is_alive()
			f2fs_get_node_page()
			f2fs_get_node_info()
		f2fs_ra_node_page() : Phase 2
		f2fs_get_read_data_page() : Phase 3
		add_gc_inode()
		find_gc_inode() : Phase 4
		move_data_page()



```c
static int gc_node_segment(struct f2fs_sb_info *sbi,
		struct f2fs_summary *sum, unsigned int segno, int gc_type)
{
	struct f2fs_summary *entry;
	block_t start_addr;
	int off;
	int phase = 0;
	bool fggc = (gc_type == FG_GC);
	int submitted = 0;
	unsigned int usable_blks_in_seg = f2fs_usable`_blks_in_seg(sbi, segno);

	start_addr = START_BLOCK(sbi, segno);

next_step:
	entry = sum;

	if (fggc && phase == 2)
		atomic_inc(&sbi->wb_sync_req[NODE]);

	for (off = 0; off < usable_blks_in_seg; off++, entry++) {
		nid_t nid = le32_to_cpu(entry->nid);
		struct page *node_page;
		struct node_info ni;
		int err;

		/* stop BG_GC if there is not enough free sections. */
		if (gc_type == BG_GC && has_not_enough_free_secs(sbi, 0, 0))
			return submitted;

		if (check_valid_map(sbi, segno, off) == 0)
			continue;

	//nid에 해당하는 NAT Entry read (for block addr of NODE)
		if (phase == 0) {
			f2fs_ra_meta_pages(sbi, NAT_BLOCK_OFFSET(nid), 1,
							META_NAT, true);
			continue;
		}
	//node page read ahead
		if (phase == 1) {
			f2fs_ra_node_page(sbi, nid);
			continue;
		}
	//node page read
		/* phase == 2 */
		node_page = f2fs_get_node_page(sbi, nid);
		if (IS_ERR(node_page))
			continue;

		/* block may become invalid during f2fs_get_node_page */
		if (check_valid_map(sbi, segno, off) == 0) {
			f2fs_put_page(node_page, 1);
			continue;
		}

		if (f2fs_get_node_info(sbi, nid, &ni, false)) {
			f2fs_put_page(node_page, 1);
			continue;
		}

		if (ni.blk_addr != start_addr + off) {
			f2fs_put_page(node_page, 1);
			continue;
		}
	//
		err = f2fs_move_node_page(node_page, gc_type);
		if (!err && gc_type == FG_GC)
			submitted++;
		stat_inc_node_blk_count(sbi, 1, gc_type);
	}

	if (++phase < 3)
		goto next_step;

	if (fggc)
		atomic_dec(&sbi->wb_sync_req[NODE]);
	return submitted;
}

```


f2fs_move_node_page()
	__write_node_page()
		f2fs_get_node_info() - lock 다수
		f2fs_down_read(&sbi->node_write)
		f2fs_do_write_node_page()
			do_write_page() - 새로운 block 할당하고 read한 data copy
				if LFS mode & Cold DATA : f2fs_down_read(&fio->sbi->io_order_lock)
				f2fs_allocate_data_block()
				f2fs_submit_page_write()
		set_node_addr() - lock 다수
		f2fs_up_read(&sbi->node_write)

