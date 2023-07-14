| Lock Type    | No. | File      | Structure           | Member name               | Comment                                                         |
| ------------ | --- | --------- | ------------------- | ------------------------- | --------------------------------------------------------------- |
| spinlock_t   | 1   | f2fs.h    | ckpt_req_control    | [stat_lock](./spinlock_t.md#spinlock_t-stat_lock)                 | lock for below checkpoint time stats                            |
| spinlock_t   | 2   | f2fs.h    | discard_cmd         | lock                      | for state updating                                              |
| spinlock_t   | 3   | f2fs.h    | extent_tree_info    | extent_lock               | locking extent lru list                                         |
| spinlock_t   | 4   | f2fs.h    | f2fs_inode_info     | i_size_lock               | protect last_disk_size                                          |
| spinlock_t   | 5   | f2fs.h    | f2fs_nm_info        | nat_list_lock             | protect clean nat entry list                                    |
| spinlock_t   | 6   | f2fs.h    | f2fs_nm_info        | nid_list_lock             | protect nid lists ops                                           |
| spinlock_t   | 7   | f2fs.h    | f2fs_bio_info       | io_lock                   | serialize DATAIOs                                               |
| spinlock_t   | 8   | f2fs.h    | inode_management    | ino_lock                  | for ino entry lock                                              |
| spinlock_t   | 9   | f2fs.h    | f2fs_sb_info        | cp_lock                   | for flag in ckpt                                                |
| spinlock_t   | 10  | f2fs.h    | f2fs_sb_info        | fsync_node_lock           | for node entry lock                                             |
| spinlock_t   | 11  | f2fs.h    | f2fs_sb_info        | inode_lock[NR_INODE_TYPE] | for dirty inode list lock (DIR_INODE / FILE_INODE / DIRTY_META) |
| spinlock_t   | 12  | f2fs.h    | f2fs_sb_info        | gc_remaining_trials_lock  |                                                                 |
| spinlock_t   | 13  | f2fs.h    | f2fs_sb_info        | [stat_lock](./spinlock_t.md#spinlock_t-stat_lock-1)                 | lock for stat operations                                        |
| spinlock_t   | 14  | f2fs.h    | f2fs_sb_info        | dev_lock                  | For multi devices & protect dirty_device                        |
| spinlock_t   | 15  | f2fs.h    | f2fs_sb_info        | error_lock                | protect errors array                                            |
| spinlock_t   | 16  | f2fs.h    | f2fs_sb_info        | iostat_lock               | #ifdef CONFIG_F2FS_IOSTAT                                       |
| spinlock_t   | 17  | f2fs.h    | f2fs_sb_info        | iostat_lat_lock           | #ifdef CONFIG_F2FS_IOSTAT                                       |
| spinlock_t   | 18  | segment.h | free_segmap_info    | segmap_lock               | free segmap lock                                                |
| mutex        | 1   | f2fs.h    | discard_cmd_control | cmd_lock                  |                                                                 |
| mutex        | 2   | f2fs.h    | extent_tree_info    | extent_tree_lock          | locking extent radix tree                                       |
| mutex        | 3   | f2fs.h    | f2fs_nm_info        | build_lock                | lock for build free nids                                        |
| mutex        | 4   | f2fs.h    | f2fs_sb_info        | writepages                | mutex for writepages()                                          |
| mutex        | 5   | f2fs.h    | f2fs_sb_info        | flush_lock                | for inode management & for flush exclusion                      |
| mutex        | 6   | f2fs.h    | f2fs_sb_info        | umount_mutex              | For shrinker support                                            |
| mutex        | 7   | segment.h | dirty_seglist_info  | seglist_lock              | lock for segment bitmaps                                        |
| mutex        | 8   | segment.h | curseg_info         | curseg_mutex              | for active log information & lock for consistency               |
| rw_semaphore | 1   | f2fs.h    | f2fs_rwsem          | internal_rwsem            | f2fs rw_sem wrapper for unfair to readers (option               |
| rw_semaphore | 2   | segment.h | sit_info            | sentry_lock               | to protect SIT cache                                            |
| rw_semaphore | 3   | segment.h | curseg_info         | [journal_rwsem](./rw_semaphore.md#rw_semaphore-journal_rwsem)             | for active log information & protect journal area               |
| f2fs_rwsem   | 1   | f2fs.h    | f2fs_inode_info     | i_sem                     | protect fi info                                                 |
| f2fs_rwsem   | 2   | f2fs.h    | f2fs_inode_info     | i_gc_rwsem[2]             | avoid racing between foreground op and gc                       |
| f2fs_rwsem   | 3   | f2fs.h    | f2fs_inode_info     | i_xattr_sem               | avoid racing between reading and changing EAs                   |
| f2fs_rwsem   | 4   | f2fs.h    | f2fs_nm_info        | nat_tree_lock             | NAT cache management & protect nat entry tree                   |
| f2fs_rwsem   | 5   | f2fs.h    | f2fs_sm_info        | curseg_lock               | for preventing curseg change                                    |
| f2fs_rwsem   | 6   | f2fs.h    | f2fs_bio_info       | io_rwsem                  | blocking op for bio                                             |
| f2fs_rwsem   | 7   | f2fs.h    | f2fs_bio_info       | bio_list_lock             | lock to protect bio entry list                                  |
| f2fs_rwsem   | 8   | f2fs.h    | f2fs_sb_info        | sb_lock                   | lock for raw super block                                        |
| f2fs_rwsem   | 9   | f2fs.h    | f2fs_sb_info        | io_order_lock             | keep migration IO order for LFS mode                            |
| f2fs_rwsem   | 10  | f2fs.h    | f2fs_sb_info        | cp_global_sem             | checkpoint procedure lock                                       |
| f2fs_rwsem   | 11  | f2fs.h    | f2fs_sb_info        | cp_rwsem                  | blocking FS operations                                          |
| f2fs_rwsem   | 12  | f2fs.h    | f2fs_sb_info        | node_write                | locking node writes                                             |
| f2fs_rwsem   | 13  | f2fs.h    | f2fs_sb_info        | node_change               | locking node change                                             |
| f2fs_rwsem   | 14  | f2fs.h    | f2fs_sb_info        | quota_sem                 | blocking cp for flags                                           |
| f2fs_rwsem   | 15  | f2fs.h    | f2fs_sb_info        | gc_lock                   | for cleaning operations                                         |
| f2fs_rwsem   | 16  | f2fs.h    | f2fs_sb_info        | pin_sem                   | for pinned files                                                |