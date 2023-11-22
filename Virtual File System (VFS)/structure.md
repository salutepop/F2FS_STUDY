
# struct kiocb
kiocb는 현재 열린 파일의 정보(ki_flip)과 해당 파일에서의 offset(ki_pos)를 갖고 있습니다.  
kiocb는 sync 및 async i/o 모두 사용되며, 아래와 같은 차이가 있습니다.
- sync i/o 경우 ki_complete = NULL
- async i/o인 경우 ki_complete = i/o가 종료된 뒤 실행 할 함수의 포인터
```c
/* (fs.h) */
struct kiocb {
	struct file *ki_filp;
	loff_t      ki_pos;
	void (*ki_complete)(struct kiocb *iocb, long ret);
	void        *private;
	int			ki_flags;
	u16			ki_ioprio; /* See linux/ioprio.h */
	struct wait_page_queue	*ki_waitq; /* for async buffered IO */
};
```

# struct iovec
iovec은 i/o에 사용할 버퍼의 정보(주소, 길이)를 담고있으며, 이름과 같이 주로 여러개의 버퍼를 배열로 담아서 사용합니다.
```c
/* (uio.h) */
struct iovec
{
	void __user *iov_base;	/* BSD uses caddr_t (1003.1g requires void *) */
	__kernel_size_t iov_len; /* Must be size_t (1003.1g) */
};
```

# struct iov_iter
iov_iter는 iovec의 정보(Read/Write op.등)를 갖고 있습니다.
- 기존 read/write() system call은 단일 buffer i/o를 처리합니다.
- iov_iter를 사용하는 read_iter/write_iter() system call은 분산된 buffer i/o를 처리하여 system call로 인한 context switching을 줄일 수 있습니다.
```c
struct iov_iter {
	u8 iter_type;
	bool copy_mc;
	bool nofault;
	bool data_source;
	bool user_backed;
	union {
		size_t iov_offset;
		int last_offset;
	};
	/*
	 * Hack alert: overlay ubuf_iovec with iovec + count, so
	 * that the members resolve correctly regardless of the type
	 * of iterator used. This means that you can use:
	 *
	 * &iter->__ubuf_iovec or iter->__iov
	 *
	 * interchangably for the user_backed cases, hence simplifying
	 * some of the cases that need to deal with both.
	 */
	union {
		/*
		 * This really should be a const, but we cannot do that without
		 * also modifying any of the zero-filling iter init functions.
		 * Leave it non-const for now, but it should be treated as such.
		 */
		struct iovec __ubuf_iovec;
		struct {
			union {
				/* use iter_iov() to get the current vec */
				const struct iovec *__iov;
				const struct kvec *kvec;
				const struct bio_vec *bvec;
				struct xarray *xarray;
				struct pipe_inode_info *pipe;
				void __user *ubuf;
			};
			size_t count;
		};
	};
	union {
		unsigned long nr_segs;
		struct {
			unsigned int head;
			unsigned int start_head;
		};
		loff_t xarray_start;
	};
};
```
`DEF_ADDRS_PER_BLOCK` Direct Node 의 address pointers 는 왜 1018인가? (1024가 아니라?)
`NIDS_PER_BLOCK` Indirect Node 의 node 갯수는 왜 1018?
> 아래 footer 때문인가?
```c
struct node_footer {
	__le32 nid;		/* node id */
	__le32 ino;		/* inode number */
	__le32 flag;		/* include cold/fsync/dentry marks and offset */
	__le64 cp_ver;		/* checkpoint version */
	__le32 next_blkaddr;	/* next node page block address */
} __packed;

struct f2fs_node {
	/* can be one of three types: inode, direct, and indirect types */
	union {
		struct f2fs_inode i;
		struct direct_node dn;
		struct indirect_node in;
	};
	struct node_footer footer;
} __packed;
```
```c
/* 200 bytes for inline xattrs by default */
#define DEFAULT_INLINE_XATTR_ADDRS	50
#define DEF_ADDRS_PER_INODE	923	/* Address Pointers in an Inode */
#define CUR_ADDRS_PER_INODE(inode)	(DEF_ADDRS_PER_INODE - \
					get_extra_isize(inode))
#define DEF_NIDS_PER_INODE	5	/* Node IDs in an Inode */
#define ADDRS_PER_INODE(inode)	addrs_per_inode(inode)
#define DEF_ADDRS_PER_BLOCK	1018	/* Address Pointers in a Direct Block */
#define ADDRS_PER_BLOCK(inode)	addrs_per_block(inode)
#define NIDS_PER_BLOCK		1018	/* Node IDs in an Indirect Block */
```

```c
struct address_space {
   struct inode            *host;          /* 해당페이지를 소유한 inode */
   struct radix_tree_root  page_tree;      /* 모든 페이지들에 대한 radix tree */
   spinlock_t              tree_lock;      /* page_tree에 대한 락 */
   unsigned int            i_mmap_writable;/* VM_SHARED 매핑 카운트 */
   struct prio_tree_root   i_mmap;         /* 모든 매핑들의 리스트 */
   struct list_head        i_mmap_nonlinear;/*VM_NONLINEAR 매핑 리스트 */
   struct mutex            i_mmap_mutex;   /* i_mmap 항목에 대한 락*/
   /* Protected by tree_lock together with the radix tree */
   unsigned long           nrpages;        /* 페이지의 총수 */
   pgoff_t                 writeback_index;/* 라이트백 시작 오프셋 */
   const struct address_space_operations *a_ops;   /* 연산테이블 */
   unsigned long           flags;          /* error bits/gfp mask */
   struct backing_dev_info *backing_dev_info; /* device readahead, etc */
   spinlock_t              private_lock;   /* for use by the address_space */
   struct list_head        private_list;   /* ditto */
   struct address_space    *assoc_mapping; /* ditto */
} __attribute__((aligned(sizeof(long))));
```

### struct f2fs_map_blocks
Now, f2fs uses f2fs_map_blocks when handling get_block.
[(LINK)](https://patchwork.kernel.org/project/linux-fsdevel/patch/1430527726-68547-5-git-send-email-jaegeuk@kernel.org/)
```c
struct f2fs_map_blocks {
	struct block_device *m_bdev;	/* for multi-device dio */
	block_t m_pblk;
	block_t m_lblk;			/* 나중에 file의 write position(시작점)이며, page offset 으로 쓰임 */
	unsigned int m_len;
	unsigned int m_flags;
	pgoff_t *m_next_pgofs;		/* point next possible non-hole pgofs */
	pgoff_t *m_next_extent;		/* point to next possible extent */
	int m_seg_type;
	bool m_may_create;		/* indicate it is from write path */
	bool m_multidev_dio;		/* indicate it allows multi-device dio */
};
```