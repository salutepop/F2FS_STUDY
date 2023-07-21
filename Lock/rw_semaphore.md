## *rw_semaphore* internal_rwsem

**`Structure`** 
```c
struct f2fs_rwsem
```

**`Initialize`** 
```c
(f2fs.h) __init_f2fs_rwsem()
```

**`Usage : WRITE`**  
f2fs_rwsem을 제어하기 위한 wrapper 함수
```c
(f2fs.h) f2fs_down_write()
(f2fs.h) f2fs_down_write_trylock()
```

**`Usage : READ`**  
f2fs_rwsem을 제어하기 위한 wrapper 함수
```c
(f2fs.h) f2fs_rwsem_is_locked()
(f2fs.h) f2fs_rwsem_is_contended()
(f2fs.h) f2fs_down_read()
(f2fs.h) f2fs_down_read_trylock()
(f2fs.h) f2fs_down_read_nested() /* #ifdef CONFIG_DEBUG_LOCK_ALLOC */
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
