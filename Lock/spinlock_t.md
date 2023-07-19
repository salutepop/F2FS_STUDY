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

## *spinlock_t* sample

**`Structure`** 
```c
Dummy
```

**`Initialize`** 
```c
Dummy
```

**`Usage`**  
Dummy
```c
Dummy
```
<br>