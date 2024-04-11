## *f2fs_rwsem* i_sem

**`Structure`** 
```c
struct f2fs_inode_info
```

**`Initialize`** 
```c
(super.c) f2fs_alloc_inode()
```

**`Usage : WRITE`**  
Dummy
```c
(dir.c) f2fs_add_regular_entry()
(dir.c) f2fs_do_tmpfile()
(dir.c) f2fs_drop_nlink()
(file.c) try_to_fix_pino()
(inline.c) f2fs_add_inline_entry()
(namei.c) f2fs_rename()
(namei.c) f2fs_cross_rename()
```

**`Usage : READ`**  
Dummy
```c
(file.c) f2fs_do_sync_file()
```
<br>

## *f2fs_rwsem* i_gc_rwsem[READ]

**`Structure`** 
```c
struct f2fs_inode_info
```

**`Initialize`** 
```c
(super.c) f2fs_alloc_inode()
```

**`Usage : WRITE`**  
Dummy
```c
(gc.c) gc_data_segment()
```

**`Usage : READ`**  
Dummy
```c
(file.c) f2fs_dio_read_iter()
(file.c) f2fs_dio_write_iter()
```
<br>

## *f2fs_rwsem* i_gc_rwsem[WRITE]

**`Structure`** 
```c
struct f2fs_inode_info
```

**`Initialize`** 
```c
(super.c) f2fs_alloc_inode()
```

**`Usage : WRITE`**  
Dummy
```c
(data.c) f2fs_write_failed()
(data.c) f2fs_migrate_blocks() /* #ifdef CONFIG_SWAP */
(file.c) f2fs_setattr()
(file.c) f2fs_punch_hole()
(file.c) f2fs_do_collapse()
(file.c) f2fs_zero_range()
(file.c) f2fs_insert_range()
(file.c) f2fs_ioc_start_atomic_write()
(file.c) f2fs_move_file_range()
(file.c) f2fs_precache_extents()
(file.c) f2fs_release_compress_blocks()
(file.c) f2fs_reserve_compress_blocks()
(file.c) f2fs_sec_trim_file()
(file.c) f2fs_file_write_iter()
(gc.c) gc_data_segment()
(segment.c) f2fs_commit_atomic_write()
(verity.c) f2fs_end_enable_verity()
```

**`Usage : READ`**  
Dummy
```c
(file.c) f2fs_dio_write_iter()
```
<br>


## *f2fs_rwsem* i_xattr_sem

**`Structure`** 
```c
struct f2fs_inode_info
```

**`Initialize`** 
```c
(super.c) f2fs_alloc_inode()
```

**`Usage : WRITE`**  
Dummy
```c
(xattr.c) f2fs_setxattr()
```

**`Usage : READ`**  
Dummy
```c
(xattr.c) f2fs_getxattr()
(xattr.c) f2fs_listxattr()
```
<br>


## *f2fs_rwsem* nat_tree_lock

**`Structure`** 
```c
struct f2fs_nm_info
```

**`Initialize`** 
```c
(node.c) init_node_manager()
```

**`Usage : WRITE`**  
Dummy
```c
(node.c) cache_nat_entry()
(node.c) set_node_addr()
(node.c) f2fs_try_to_free_nats()
(node.c) f2fs_flush_nat_entries()
(node.c) f2fs_destroy_node_manager()
```

**`Usage : READ`**  
Dummy
```c
(node.c) f2fs_need_dentry_mark()
(node.c) f2fs_is_checkpointed_node()
(node.c) f2fs_need_inode_block_update()
(node.c) f2fs_get_node_info()
(node.c) f2fs_nat_bitmap_enabled()
(node.c) scan_free_nid_bits()
(node.c) __f2fs_build_free_nids()
(node.c) f2fs_enable_nat_bits()
```
<br>


## *f2fs_rwsem* curseg_lock

**`Structure`** 
```c
struct f2fs_sm_info
```

**`Initialize`** 
```c
(segment.c) f2fs_build_segment_manager()
```

**`Usage : WRITE`**  
Dummy
```c
(segment.c) f2fs_do_replace_block()
```

**`Usage : READ`**  
Dummy
```c
(segment.c) __f2fs_init_atgc_curseg()
(segment.c) f2fs_allocate_segment_for_resize()
(segment.c) f2fs_allocate_new_section()
(segment.c) f2fs_allocate_new_segments()
(segment.c) f2fs_allocate_data_block()
```
<br>


## *f2fs_rwsem* io_rwsem

**`Structure`** 
```c
struct f2fs_bio_info
```

**`Initialize`** 
```c
(data.c) f2fs_init_write_merge_io()
```

**`Usage : WRITE`**  
Dummy
```c
(data.c) __f2fs_submit_merged_write()
(data.c) f2fs_submit_page_write()
```

**`Usage : READ`**  
Dummy
```c
(data.c) __submit_merged_write_cond()
```
<br>


## *f2fs_rwsem* bio_list_lock

**`Structure`** 
```c
struct f2fs_bio_info
```

**`Initialize`** 
```c
(data.c) f2fs_init_write_merge_io()
```

**`Usage : WRITE`**  
Dummy
```c
(data.c) add_bio_entry()
(data.c) add_ipu_page()
```

**`Usage : READ`**  
Dummy
```c
(data.c) f2fs_submit_merged_ipu_write()
```
<br>


## *f2fs_rwsem* sb_lock

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(super.c) init_sb_info()
```

**`Usage : WRITE`**  
Dummy
```c
(file.c) f2fs_ioc_get_encryption_pwsalt()
(file.c) f2fs_ioc_setfslabel()
(gc.c) update_sb_metadata()
(super.c) f2fs_handle_stop()
(super.c) f2fs_handle_error()
(sysfs.c) __sbi_store()
```

**`Usage : READ`**  
Dummy
```c
(file.c) f2fs_ioc_getfslabel()
(namei.c) set_compress_new_inode()
(namei.c) set_file_temperature()
```
<br>


## *f2fs_rwsem* io_order_lock

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(super.c) init_sb_info()
```

**`Usage : WRITE`**  
Dummy
```c
(gc.c) move_data_block()
```

**`Usage : READ`**  
Dummy
```c
(segment.c) do_write_page()
```
<br>


## *f2fs_rwsem* cp_global_sem

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(super.c) f2fs_fill_super()
```

**`Usage : WRITE`**  
Dummy
```c
(checkpoint.c) f2fs_write_meta_pages()
(checkpoint.c) f2fs_write_checkpoint()
(gc.c) f2fs_resize_fs()
(recovery.c) f2fs_recover_fsync_data()
```

**`Usage : READ`**  
read lock은 아니고, f2fs_rwsem_is_locked() 사용
```c
(node.c) cache_nat_entry() /* f2fs_rwsem_is_locked() */
```
<br>


## *f2fs_rwsem* cp_rwsem

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(super.c) f2fs_fill_super()
```

**`Usage : WRITE`**  
Dummy
```c
(f2fs.h) f2fs_lock_all()
    > (checkpoint.c) block_operations()
```

**`Usage : READ`**  
Dummy
```c
(f2fs.h) f2fs_lock_op()
    > (data.c) f2fs_map_lock()
    > (data.c) f2fs_migrate_blocks() /* #ifdef CONFIG_SWAP */
    > (file.c) f2fs_do_truncate_blocks()
    > (file.c) f2fs_setattr()
    > (file.c) fill_zero()
    > (file.c) f2fs_punch_hole()
    > (file.c) f2fs_do_collapse()
    > (file.c) f2fs_zero_range()
    > (file.c) f2fs_insert_range()
    > (file.c) f2fs_expand_inode_data()
    > (file.c) f2fs_move_file_range()
    > (file.c) f2fs_ioc_setproject() /* #ifdef CONFIG_QUOTA */
    > (gc.c) f2fs_resize_fs()
    > (inline.c) f2fs_convert_inline_inode()
    > (inline.c) f2fs_try_convert_inline_dir()
    > (inode.c) f2fs_evict_inode()
    > (namei.c) f2fs_create()
    > (namei.c) f2fs_link()
    > (namei.c) __recover_dot_dentries()
    > (namei.c) f2fs_unlink()
    > (namei.c) f2fs_symlink()
    > (namei.c) f2fs_mkdir()
    > (namei.c) f2fs_mknod()
    > (namei.c) __f2fs_tmpfile()
    > (namei.c) f2fs_rename()
    > (namei.c) f2fs_cross_rename()
    > (segment.c) f2fs_commit_atomic_write()
    > (super.c) f2fs_quota_sync() /* #ifdef CONFIG_QUOTA */
    > (xattr.c) f2fs_setxattr()

(f2fs.h) f2fs_trylock_op()
    > (compress.c) f2fs_write_compressed_pages()
    > (data.c) f2fs_do_write_data_page()

(segment.c) excess_dirty_threshold() /* f2fs_rwsem_is_locked */
    > (segment.c) f2fs_balance_fs_bg()
```
<br>


## *f2fs_rwsem* node_write

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(super.c) f2fs_fill_super()
```

**`Usage : WRITE`**  
Dummy
```c
(checkpoint.c) block_operations()
```

**`Usage : READ`**  
Dummy
```c
(compress.c) f2fs_write_compressed_pages()
(data.c) f2fs_write_single_data_page()
(node.c) __write_node_page()
```
<br>


## *f2fs_rwsem* node_change

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(super.c) f2fs_fill_super()
```

**`Usage : WRITE`**  
Dummy
```c
(checkpoint.c) block_operations()
```

**`Usage : READ`**  
Dummy
```c
(data.c) f2fs_map_lock()
```
<br>


## *f2fs_rwsem* quota_sem

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(super.c) f2fs_fill_super()
```

**`Usage : WRITE`**  
Dummy
```c
(checkpoint.c) __need_flush_quota()
```

**`Usage : READ`**  
Dummy
```c
(super.c) f2fs_quota_sync() /* #ifdef CONFIG_QUOTA */
(super.c) f2fs_dquot_commit() /* #ifdef CONFIG_QUOTA */
(super.c) f2fs_dquot_acquire() /* #ifdef CONFIG_QUOTA */
```
<br>


## *f2fs_rwsem* gc_lock

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(super.c) f2fs_fill_super()
```

**`Usage : WRITE`**  
Dummy
```c
(checkpoint.c) __write_checkpoint_sync()
(checkpoint.c) f2fs_issue_checkpoint()
(file.c) f2fs_expand_inode_data()
(file.c) f2fs_ioc_gc()
(file.c) __f2fs_ioc_gc_range()
(file.c) f2fs_ioc_flush_device()
(gc.c) gc_thread_func()
(gc.c) f2fs_resize_fs()
(segment.c) f2fs_balance_fs()
(segment.c) f2fs_trim_fs()
(super.c) f2fs_disable_checkpoint()
(super.c) f2fs_enable_checkpoint()
```

**`Usage : READ`**  
read lock 사용처 확인 안됨
```c
Dummy
```
<br>


## *f2fs_rwsem* pin_sem

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(super.c) init_sb_info()
```

**`Usage : WRITE`**  
Dummy
```c
(data.c) f2fs_migrate_blocks() /* #ifdef CONFIG_SWAP */
(file.c) f2fs_expand_inode_data()
```

**`Usage : READ`**  
read lock 사용처 확인 안됨
```c
Dummy
```
<br>