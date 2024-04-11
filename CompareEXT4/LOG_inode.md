### Initial
`mkfs/f2fs_format.c`
*ino 추가하고, next_free_nid 한개 증가시키기*
set_sb(node_ino, 1);
set_sb(meta_ino, 2);
set_sb(root_ino, 3);
c.next_free_nid = 4;


`include/linux/f2fs_fs.h`
*RESERVED_NUM 4로 변경*
/* 0, 1(node nid), 2(meta nid) are reserved node id */
#define F2FS_RESERVED_NODE_NUM      3

`super.c - init_sb_info()`
F2FS_META_INO(sbi) = le32_to_cpu(raw_super->meta_ino);

`super.c - f2fs_fill_super()`
/* get an inode for meta space */
sbi->meta_inode = f2fs_iget(sb, F2FS_META_INO(sbi));
if (IS_ERR(sbi->meta_inode)) {
    f2fs_err(sbi, "Failed to read F2FS meta data inode");
    err = PTR_ERR(sbi->meta_inode);
    goto free_page_array_cache;
}

`f2fs.h`
static inline struct address_space *META_MAPPING(struct f2fs_sb_info *sbi)
{
    return sbi->meta_inode->i_mapping;
}


### Free
`super.c - f2fs_put_super()`
if (err) {
    truncate_inode_pages_final(NODE_MAPPING(sbi));
    truncate_inode_pages_final(META_MAPPING(sbi));
}

`super.c - f2fs_fill_super()`
free_mate:
    truncate_inode_pages_final(META_MAPPING(sbi));


### Write
1) LOG page cache에 buffered write, end CP 에서 LOG -> NODE로 mem copy
   - f2fs_buffered_write_iter()
     - generic_perform_write(iocb, from)
    meta 정보는 어떻게 하지, 4kb page 앞에 inode/nid 등을 적어버리면 4kb 전체를 쓸수없어지면서 4kb전체를 쓰는 경우 필요한 page 수가 많아지게 되는데, 다루기가 힘들 듯 함

