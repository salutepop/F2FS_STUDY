## *spinlock_t* stat_lock

**`Structure`** 
```cpp
struct ckpt_req_control 
```

**`Initialize`** 
```cpp
(checkpoint.c) f2fs_init_ckpt_req_control()
```

**`Usage`**  
Check point time 변수(Current/Peak) 접근하는 경우
```cpp
(checkpoint.c) __checkpoint_and_complete_reqs()
(debug.c) update_general_status()
```
<br>

## *spinlock_t* stat_lock

**`Structure`** 
```cpp
struct f2fs_sb_info 
```

**`Initialize`** 
```cpp
(super.c) f2fs_fill_super() 
```

**`Usage`**  
sbi의 block count 변수(unusable/valid/reserve 등)에 접근하는 경우
```cpp
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
```cpp
Dummy
```

**`Initialize`** 
```cpp
Dummy
```

**`Usage`**  
Dummy
```cpp
Dummy
```
<br>