## *rw_semaphore* journal_rwsem

**`Structure`** 
```c
struct curseg_info /* for active log information */
```

**`Initialize`** 
```c
(segment.c) build_curseg()
```

**`Usage : WRITE`**  
current segment에서 cache된(journal) data를(nat/sit 등) 수정/삭제 할 때
```c
(node.c) remove_nats_in_journal()
(node.c) __flush_nat_entry_set()
(segment.c) read_normal_summaries() /* update journal info */
(segment.c) remove_sits_in_journal()
(segment.c) f2fs_flush_sit_entries()
```

**`Usage : READ`**  
current segment에서 cache된(journal) data를(nat/sit 등) 찾을 때
```c
(node.c) f2fs_get_node_info()
(node.c) scan_curseg_cache()
(segment.c) write_current_sum_page()
(segment.c) build_sit_entries()
```
<br>


## *rw_semaphore* sentry_lock

**`Structure`** 
```c
struct sit_info
```

**`Initialize`** 
```c
(segment.c) build_sit_info()
```

**`Usage : WRITE`**  
Dummy
```c
(gc.c) __get_victim()
(segment.c) f2fs_invalidate_blocks()
(segment.c) __f2fs_init_atgc_curseg()
(segment.c) f2fs_allocate_segment_for_resize()
(segment.c) f2fs_allocate_new_section()
(segment.c) f2fs_allocate_new_segments()
(segment.c) f2fs_exist_trim_candidates()
(segment.c) f2fs_allocate_data_block()
(segment.c) f2fs_do_replace_block()
(segment.c) f2fs_flush_sit_entries()
(segment.c) init_min_max_mtime()
```

**`Usage : READ`**  
Dummy
```c
(gc.c) check_valid_map()
(segment.c) f2fs_is_checkpointed_data()
```
<br>

## *rw_semaphore* journal_rwsem

**`Structure`** 
```c
struct curseg_info
```

**`Initialize`** 
```c
(segment.c) build_curseg()
```

**`Usage : WRITE`**  
Dummy
```c
(node.c) remove_nats_in_journal()
(node.c) __flush_nat_entry_set()
(segment.c) read_normal_summaries()
(segment.c) remove_sits_in_journal()
(segment.c) f2fs_flush_sit_entries()
```

**`Usage : READ`**  
Dummy
```c
(node.c) f2fs_get_node_info()
(node.c) scan_curseg_cache()
(segment.c) write_current_sum_page()
(segment.c) build_sit_entries()
```
<br>
