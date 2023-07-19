## *mutex* cmd_lock

**`Structure`** 
```c
struct discard_cmd_control
```

**`Initialize`** 
```c
(segment.c) create_discard_cmd_control()
```

**`Usage`**  
* discard_cmd_control 멤버 접근 시 사용
* 특히, discard command를 위한 rb_tree 수정/삭제 시 사용
```c
(segment.c) __queue_zone_reset_cmd() //CONFIG_BLK_DEV_ZONED
(segment.c) __queue_discard_cmd()
(segment.c) __issue_discard_cmd_orderly()
(segment.c) __issue_discard_cmd()
(segment.c) __drop_discard_cmd()
(segment.c) __wait_one_discard_bio()
(segment.c) __wait_discard_cmd_range()
(segment.c) f2fs_wait_discard_bio()
(segment.c) __issue_discard_cmd_range()
(segment.h) wake_up_discard_thread()
(sysfs.c) discard_plist_seq_show()
```
<br>

## mutex extent_tree_lock

**`Structure`** 
```c
struct extent_tree_info
```

**`Initialize`** 
```c
(extent_cache.c) __init_extent_tree_info()
```

**`Usage`**  
* extent_tree list 및 zombie tree list 수정/삭제 시 사용
* dnode page cache와 disk 간의 cache 효율(cache hit)을 높이기 위해 rb_tree기반의 extent_cache 가 사용됩니다.

```c
(extent_cache.c) __grab_extent_tree()
(extent_cache.c) __shrink_extent_tree()
(extent_cache.c) __destroy_extent_tree()
(extent_cache.c) __destroy_extent_tree()
```
<br>

## mutex build_lock

**`Structure`** 
```c
struct f2fs_nm_info
```

**`Initialize`** 
```c
(node.c) init_node_manager()
```

**`Usage`**  
Dummy
```c
(node.c) f2fs_alloc_nid()
(node.c) f2fs_build_free_nids()
(node.c) f2fs_try_to_free_nids()
```
<br>

## mutex writepages

**`Structure`** 
```c
struct f2fs_sb_info
```

**`Initialize`** 
```c
(super.c) f2fs_fill_super()
```

**`Usage`**  
* Serialize I/O 상황에서 page cache에 data write할 때 사용
* __should_serialize_io() 참고
```c
(data.c) __f2fs_write_data_pages()
```
<br>

## mutex sample

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

## mutex sample

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