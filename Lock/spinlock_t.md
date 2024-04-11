## *spinlock_t* stat_lock

**`Structure`** 
```c
struct ckpt_req_control 
```

**`Initialize`** 
```c
(checkpoint.c) f2fs_init_ckpt_req_control()
```

**`Usage`**  
Check point time 변수(Current/Peak) 접근하는 경우
```c
(checkpoint.c) __checkpoint_and_complete_reqs()
(debug.c) update_general_status()
```
<br>

## *spinlock_t* discard_cmd

**`Structure`** 
```c
(f2fs.h) struct discard_cmd
```

**`Initialize`**
```c
(segment.c) __create_discard_cmd()
```

**`Usage`**  
Discard command 생성/수정/삭제 과정에서 state 혹은 bio_ref 접근 시 사용
* discard command issue 시, state=D_SUBMIT, bio_ref == 1
* discard command complete 시, state=D_DONE, bio_ref == 0
```c
(segment.c) __remove_discard_cmd()
(segment.c) f2fs_submit_discard_endio()
(segment.c) __submit_discard_cmd()
```
<br>

## *spinlock_t* extent_tree_info

**`Structure`** 
```c
(f2fs.h) struct extent_tree_info
```

**`Initialize`** 
```c
(extent_cache.c) __init_extent_tree_info()
```

**`Usage`**  
extent_node lis (&en->list) 및 extent_tree_info list (&eti->extent_list) 접근 시 사용
```c
__release_extent_node()
f2fs_init_read_extent_tree()
__lookup_extent_tree()
__try_merge_extent_node()
__insert_extent_tree()
__shrink_extent_tree()
```
<br>

## *spinlock_t* i_size_lock

**`Structure`** 
```c
(f2fs.h) struct f2fs_inode_info
```

**`Initialize`** 
```c
(super.c) f2fs_alloc_inode()
```

**`Usage`**  
Dummy
```c
(compress.c) f2fs_write_compressed_pages()
(data.c) f2fs_write_single_data_page()
(f2fs.h) f2fs_skip_inode_update()
(file.c) f2fs_setattr()
```
<br>

## *spinlock_t* nat_list_lock

**`Structure`** 
```c
struct f2fs_nm_info
```

**`Initialize`** 
```c
(node.c) init_node_manager()
```

**`Usage`**  
NAT Entry를 제어하는 경우
* nat entry 삽입 시
* ne_list 초기화/삭제 시
* nat cache list 삭제 시, nat entry도 삭제(free)하기 위해

```c
(node.c) __init_nat_entry()
(node.c) __lookup_nat_cache()
(node.c) __set_nat_cache_dirty()
(node.c) __clear_nat_cache_dirty()
(node.c) f2fs_try_to_free_nats()
(node.c) f2fs_destroy_node_manager()
```
<br>

## *spinlock_t* nid_list_lock

**`Structure`** 
```c
struct f2fs_nm_info
```

**`Initialize`** 
```c
(node.c) init_node_manager()
```

**`Usage`**  
free node id list를 제어하는 경우 및 available_nids 변수에 접근할 때 사용
nid 할당/삭제 할 때마다 free nid list update 됨 (update_free_nid_bitmap)
```c
(node.c) add_free_nid()
(node.c) remove_free_nid()
(node.c) scan_nat_page()
(node.c) f2fs_alloc_nid()
(node.c) f2fs_alloc_nid_done()
(node.c) f2fs_alloc_nid_failed()
(node.c) f2fs_try_to_free_nids()
(node.c) remove_nats_in_journal()
(node.c) __flush_nat_entry_set()
(node.c) load_free_nid_bitmap()
(node.c) f2fs_destroy_node_manager()
(node.h) next_free_nid()
```
<br>

## *spinlock_t* io_lock
>**mutex lock -> fifo list spinlock(io_lock)으로 개선된 이력이 있음**  
>f2fs: introduce io_list for serialize data/node IOs
>
>Serialize data/node IOs by using fifo list instead of mutex lock,  
>it will help to enhance concurrency of f2fs, meanwhile keeping LFS  
>IO semantics.
>
>Signed-off-by: Chao Yu  
>Signed-off-by: Jaegeuk Kim 

**`Structure`** 
```c
struct f2fs_bio_info
```

**`Initialize`** 
```c
(data.c) f2fs_init_write_merge_io()
```

**`Usage`**  
*목적은 잘 모르겠지만,* Data/Node io를 serialize 하기 위해 사용 되는 lock으로  
fio의 list인 io_list를 접근하는 경우 사용 됨
```c
(data.c) f2fs_submit_page_write()
(segment.c) f2fs_allocate_data_block()
```
<br>

## *spinlock_t* ino_lock

**`Structure`** 
```c
struct inode_management
```

**`Initialize`** 
```c
(checkpoint.c) f2fs_init_ino_entry_info()
```

**`Usage`**  
Dummy
```c
(checkpoint.c) __add_ino_entry()
(checkpoint.c) __remove_ino_entry()
(checkpoint.c) f2fs_exist_written_data()
(checkpoint.c) f2fs_release_ino_entry()
(checkpoint.c) f2fs_is_dirty_device()
(checkpoint.c) f2fs_acquire_orphan_inode()
(checkpoint.c) f2fs_release_orphan_inode()
```
<br>

## *spinlock_t* cp_lock

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(super.c) init_sb_info()
```

**`Usage`**  
Dummy
```c
(checkpoint.c) update_ckpt_flags()
(checkpoint.c) do_checkpoint()
(f2fs.h) set_ckpt_flags()
(f2fs.h) clear_ckpt_flags()
```
<br>

## *spinlock_t* fsync_node_lock

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(node.c) f2fs_init_fsync_node_info()
```

**`Usage`**  
Dummy
```c
(node.c) f2fs_add_fsync_node_entry()
(node.c) f2fs_del_fsync_node_entry()
(node.c) f2fs_reset_fsync_node_info()
(node.c) f2fs_wait_on_node_pages_writeback()
```
<br>

## *spinlock_t* inode_lock

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(super.c) f2fs_fill_super()
```

**`Usage`**  
Dummy
```c
(checkpoint.c) f2fs_update_dirty_folio()
(checkpoint.c) f2fs_remove_dirty_inode()
(checkpoint.c) f2fs_sync_dirty_inodes()
(checkpoint.c) f2fs_sync_inode_meta()
(f2fs.h) f2fs_skip_inode_update()
(node.c) f2fs_match_ino()
(super.c) f2fs_inode_dirtied()
(super.c) f2fs_inode_synced()
```
<br>

## *spinlock_t* gc_remaining_trials_lock

**`Structure`** 
```c
struct f2fs_sb_info 
```

**`Initialize`** 
```c
(super.c) init_sb_info()
```

**`Usage`**  
Dummy
```c
(gc.c) gc_thread_func()
(sysfs.c) __sbi_store()
```
<br>

## *spinlock_t* stat_lock

**`Structure`** 
```c
struct f2fs_sb_info 
```

**`Initialize`** 
```c
(super.c) f2fs_fill_super() 
```

**`Usage`**  
sbi의 block count 변수(unusable/valid/reserve 등)에 접근하는 경우
```c
(checkpoint.c) do_checkpoint()
(f2fs.h) inc_valid_block_count()
(f2fs.h) dec_valid_block_count()
(f2fs.h) inc_valid_node_count()
(f2fs.h) dec_valid_node_count()
(gc.c) f2fs_resize_fs()
(segment.c) update_sit_entry()
(super.c) f2fs_statfs()
(super.c) f2fs_disable_checkpoint()
```
<br>

## *spinlock_t* dev_lock

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(super.c) init_sb_info()
```

**`Usage`**  
Dummy
```c
(segment.c) f2fs_flush_device_cache()
(segment.c) f2fs_update_device_state()
```
<br>

## *spinlock_t* error_lock

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(super.c) f2fs_fill_super()
```

**`Usage`**  
Dummy
```c
(super.c) f2fs_save_errors()
(super.c) f2fs_update_errors()
```
<br>

## *spinlock_t* iostat_lock

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(iostat.c) f2fs_init_iostat()
```

**`Usage`**  
Dummy
```c
(iostat.c) f2fs_record_iostat()
(iostat.c) f2fs_reset_iostat()
(iostat.c) f2fs_update_iostat()
(sysfs.c) __sbi_store()
```
<br>

## *spinlock_t* iostat_lat_lock

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(iostat.c) f2fs_init_iostat()
```

**`Usage`**  
Dummy
```c
(iostat.c) __record_iostat_latency()
(iostat.c) f2fs_reset_iostat()
(iostat.c) __update_iostat_latency()
```
<br>

## *spinlock_t* segmap_lock

**`Structure`** 
```c
struct free_segmap_info
```

**`Initialize`** 
```c
(segment.c) build_free_segmap()
```

**`Usage`**  
Dummy
```c
(gc.h) free_segs_blk_count_zoned()
(segment.c) get_new_segment()
(segment.h) find_next_inuse()
(segment.h) __set_free()
(segment.h) __set_test_and_free()
(segment.h) __set_test_and_inuse()
```
<br>