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


## *rw_semaphore* Dummy

**`Structure`** 
```c
Dummy
```

**`Initialize`** 
```c
Dummy
```

**`Usage : WRITE`**  
Dummy
```c
Dummy
```

**`Usage : READ`**  
Dummy
```c
Dummy
```
<br>
