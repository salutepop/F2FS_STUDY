1. 첫번째 CP 오래 걸리는 이유
   1. background GC 이지만 gc_thread에 의해 주기적(60초) 동작하는 BG GC 아니고, f2fs_write_node_pages에 의해 호출
   2. 30배 누적되어서 30배 오래걸리는 것임.
      1. file create 약 10만번쯤 호출 시점에 첫 CP 이벤트 발생
      2. 이후 부터는 약 3천번마다 CP 이벤트 발생
      3. create 호출당으로 환산하면 비슷함.
      4. write_checkpoint는 write back이 필요하기 때문에 device 성능에 연관
2. f2fs_write_checkpoint 수행 시간 줄이는 방법 검토
   1. 함수 profiling 수행 시, block operation 소요시간 오래 걸림 -> ftrace로 block ops 시간 확인
      ```java
      f2fs_ckpt-259:0-155688  [006] ..... 410642.226044: f2fs_write_checkpoint: dev = (259,0), checkpoint for Sync, state = start block_ops
      f2fs_ckpt-259:0-155688  [008] ..... 410642.677723: f2fs_write_checkpoint: dev = (259,0), checkpoint for Sync, state = finish block_ops
      f2fs_ckpt-259:0-155688  [008] ..... 410642.703924: f2fs_write_checkpoint: dev = (259,0), checkpoint for Sync, state = finish checkpoint
      f2fs_ckpt-259:0-155688  [008] ..... 410642.703926: f2fs_write_checkpoint: dev = (259,0), checkpoint for Sync, state = elapsed 477882954
      ```
   2. write_checkpoint할 때, write_back behavior 분석
      1. read는 연속된 lba임에도, lba단위(4kB)로 반복됨
      2. write는 연속되기도하고 안되기도하지만, len=31(128kB) 단위로 써짐




f2fs_flush_inline_data
f2fs_quota_sync


checkpoint 우회 (cp flag가 set 되어있으면 새로운 cp는 wait? drop?)
1. mem copy (메모리 두배 사용 in worst case)
   1. set cp flag
   2. lock_all // cp_rwsem
   3. CP 관련 메모리를 복사 (page cache는 어떡하지, cp 중 write 동작으로 업데이트 될 것 같은데)
      1. page cache까지 복사하면 해결 되지만(하나하나 탐색/복사 비용),
      2. page cache를 복사하지 않으려면
         1. file create 워크로드는 생성 직후에만 매우 작은 양을 write하기 때문에 파일간의 page cache가 겹치진 않을 듯 함
         2. 만약 겹치는 경우엔 i/o처리를 지연(대기)시켜야 할듯
   4. unlock all
   5. 복사된 메모리를 활용하여 CP 수행 + 정상적으로 i/o 처리
   6. CP 완료 되면, sbi->ckpt 정보 업데이트
   7. reset cp flag
   <예상 문제점>
   cp 중간에 cp 요청 들어오는 경우 : 무시(skip), bg gc 정책도 동일

   
2. non-blocking cp 수행 + log block
   1. set cp flag
   2. 정상적으로 i/o 요청을 받을 수 있게 lock 최대한 배제하여 cp 수행
   3. cp flag가 set되어있으면 i/o 처리를 위해 log block 사용
      1. read (신경쓸 필요는 없을듯)
         1. log block 탐색
         2. 없으면 page cache 탐색 (CP 동작 중 lock 걸릴 수도?)
      2. write
         1. log block에 write
   4. cp 완료 후 read 데이터는 무시하고, log block 정보는 write수행
   5. reset cp flag


struct f2fs_rwsem cp_global_sem;    /* checkpoint procedure lock */
struct f2fs_rwsem cp_rwsem;     /* blocking FS operations */
   > (read lock) map_lock / convert_inline_inode / f2fs_create ..
struct f2fs_rwsem node_write;       /* locking node writes */
   > (read lock) write_node_page / write_single_data_pate
struct f2fs_rwsem node_change;  /* locking node change */
   > (read_lock) map_lock


cp recovery과정에서 data도 보장해주나?

struct inode *node_inode;       /* cache node blocks */
   - index = nid (file offset / data offset-LBA)
struct inode *meta_inode;       /* cache meta blocks */ 
   - F2FS CP/NAT/SIt/SSA 모두 커버, index = lba
LOG 보다는 COW로 처리하는것


// write semaphore : &sbi->cp_rwsem, &sbi->node_write
- f2fs_write_checkpoint()
   - block_operations()
     - f2fs_flush_inline_data(sbi)
       - NODE_MAPPING(sbi)
     - f2fs_quota_sync_file(sbi, cnt)  // cnt < MAXQUOTAS
       - struct quota_info *dqopt = sb_dqopt(sbi->sb)
       - dqopt->files[type]->i_mapping
     - f2fs_sync_dirty_inodes(sbi, DIR_INODE, true)
       - &sbi->inode_list[DIR_INODE]
     - f2fs_sync_node_pages()
       - NODE_MAPPING(sbi)
     - ETC
       - F2FS_CKPT(sbi)
       - NM_I(sbi)
    
   - f2fs_flush_merged_writes()
     - sbi->write_io[btype]
  
   - f2fs_flush_nat_entries()
     - NM_I(sbi)
     - CURSEG_I(sbi, CURSEG_HOT_DATA), curseg->journal
     - f2fs_down_write(&nm_i->nat_tree_lock)
  
   - f2fs_flush_sit_entries()
     - SIT_I(sbi), &sit_i->sentry_lock
     - CURSEG_I(sbi, CURSEG_COLD_DATA), curseg->journal
     - &SM_I(sbi)->sit_entry_set
     - test_bit(type, &sbi->s_flag)
     - add_sits_in_set()
       - &sm_info->sit_entry_set
       - SIT_I(sbi)->dirty_sentries_bitmap
     - 
   - f2fs_save_inmem_curseg()
     - &DIRTY_I(sbi)->seglist_lock
     - CURSEG_I(sbi, CURSEG_COLD_DATA_PINNED)
     - CURSEG_I(sbi, CURSEG_ALL_DATA_ATGC)
  
   - do_checkpoint()
     - F2FS_CKPT(sbi)
     - NM_I(sbi)
     - 
   - f2fs_clear_prefree_segments()
   - f2fs_restore_inmem_curseg()
   - unblock_operations()