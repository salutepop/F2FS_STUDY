## *rw_semaphore* journal_rwsem

**`Structure`** 
```cpp
struct curseg_info /* for active log information */
```

**`Initialize`** 
```cpp
(segment.c) build_curseg()
```

**`Usage : WRITE`**  
current segment에서 cache된(journal) data를(nat/sit 등) 수정/삭제 할 때
```cpp
(node.c) remove_nats_in_journal()
(node.c) __flush_nat_entry_set()
(segment.c) read_normal_summaries() /* update journal info */
(segment.c) remove_sits_in_journal()
(segment.c) f2fs_flush_sit_entries()
```

**`Usage : READ`**  
current segment에서 cache된(journal) data를(nat/sit 등) 찾을 때
```cpp
(node.c) f2fs_get_node_info()
(node.c) scan_curseg_cache()
(segment.c) write_current_sum_page()
(segment.c) build_sit_entries()
```
<br>


## *rw_semaphore* Dummy

**`Structure`** 
```cpp
Dummy
```

**`Initialize`** 
```cpp
Dummy
```

**`Usage : WRITE`**  
Dummy
```cpp
Dummy
```

**`Usage : READ`**  
Dummy
```cpp
Dummy
```
<br>
