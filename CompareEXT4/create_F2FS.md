f2fs_create (19.7)
    f2fs_new_inode  (5.2)
        f2fs_init_extent_tree   (0.6)
    f2fs_do_add_link    (7.5)
        f2fs_setup_filename (0.2)
            __f2fs_setup_filename   (0.4)
        __f2fs_find_entry   (1.2)
            f2fs_find_in_inline_dir (1)
        f2fs_add_dentry (7.2)
            f2fs_add_inline_entry   (6.2)
                f2fs_init_inode_metadata    (3.7)
                    f2fs_new_inode_page (3.0)
                        f2fs_new_node_page  (2.9)
                f2fs_update_inode_page  (1.1)
                f2fs_update_parent_metadata (1.8)
            f2fs_add_regular_entry  (0.8)
    f2fs_alloc_nid_done (0.3)
    f2fs_balance_fs (6.3)
        f2fs_balance_fs_bg  (5.2)
            f2fs_try_to_free_nats   (0.1)
            f2fs_build_free_nids    (0.3)
            f2fs_sync_fs    (3.5)
                f2fs_issue_checkpoint   (3.5)

Thread 1 기준
f2fs_create                     (12.9)
    .. 나머지 생략
    f2fs_balance_fs                 (5.3)
        f2fs_balance_fs_bg              (4.5)
            f2fs_available_free_memory      (0.0)
            f2fs_shrink_read_extent_tree    (0.0)
            f2fs_shrink_age_extent_tree     (-)
            f2fs_try_to_free_nats           (0.1)
            f2fs_try_to_free_nids           (-)
            f2fs_build_free_nids            (0.2)
            f2fs_sync_dirty_inodes          (0.0)
            f2fs_sync_fs                    (4.2)
                f2fs_issue_checkpoint           (4.2)
        f2fs_gc                         (-)


2.614: Running...
25.618: Run took 23 seconds...
createfile1          1000000ops    43472ops/s   0.0mb/s    0.020ms/op [0.011ms - 541.206ms]
