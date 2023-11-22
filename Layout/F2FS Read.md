### Overview
## Read Process
1. vfs_read() : f2fs_file_read_iter()
2. filemap_read() : 
3. do ~ while(iov_iter_count(iter))
   1. filemap_get_pages() : Page cache에서 페이지를 가져옴, 만약 page cache에 없으면 storage에서 읽어옴
      1. filemap_get_read_batch() : page cache에서 page(folio)를 가져와서 fbatch에 추가
      2. page_cache_sync_readahead() : page cache에서 읽어오지 못하면, storage로 부터 page cache로 data read 한 뒤, 다시 filemap_get_read_batch() 를 호출하여 page cache로 부터 page를 가져와서 fbatch에 추가
         1. read_pages() : `aops->readahead` 를 통해 storage에서 data를 읽어옴
      3. filemap_readahead() : readahead가 설정되어있는 경우 last index로 부터 추가로 data를 읽어옴. (page_cache_async_ra()) \
      [Sync/Async Readahead 추가설명](https://lwn.net/Articles/888715/)

## Readahead Process
1. f2fs_readahead() :
   1. f2fs_mpage_readpages()
      1. f2fs_read_single_page()
         1. f2fs_map_blocks() : 필요한 map blocks의 길이를 dnode_of_data구조체로 할당받아옴 
         2. f2fs_grab_read_bio() : read를 위해 제출 할 bio 설정
            1. bio_alloc_bioset() : mempool로 부터 bio를 할당 받음
      2. f2fs_submit_read_bio()


### f2fs_readahead()
f2fs_mpage_readpages() \
1. for page in pages { f2fs_read_single_page() }
2. f2fs_submit_read_bio()

### f2fs_read_single_page()
1. f2fs_map_blocks() : 필요한 map blocks의 길이를 할당받아옴 dnode_of_data구조체로
2. f2fs_grab_read_bio()
3. bio_add_page()

### f2fs_submit_read_bio()
submit_bio()


bio->bi_end_io = f2fs_read_end_io;

## Read
```
f2fs_file_read_iter()
	filemap_read()
	while(iov_iter_count(iter))
		filemap_get_pages() : Page cache에서 페이지를 가져옴, 만약 page cache에 없으면 storage에서 읽어옴
			filemap_get_read_batch() : page cache에서 page(folio)를 가져와서 fbatch에 추가
			page_cache_sync_readahead() : page cache에서 읽어오지 못하면, storage로 부터 page cache로 data read
				read_pages() : `aops->readahead` 를 통해 storage에서 data를 읽어옴
				== f2fs_readahead()
					f2fs_mpage_readpages()
					for page in pages
						f2fs_read_single_page()
							f2fs_map_blocks() : 
								f2fs_get_dnode_of_data() - lock_page(page)
							if(bio is NULL) 
								f2fs_grab_read_bio() : mempool에서 bio 할당, bi_end_io = f2fs_read_end_io
							bio_add_page() : bio에 page추가, 필요 시 page 단위 merge
							if(bio is full) 
								f2fs_submit_read_bio()
									f2fs_read_end_io() : bio의 pages unlock하고, bio 메모리 해제
			filemap_get_read_batch() : storage 로 읽은 data를, page cache에서 page(folio)를 가져와서 fbatch에 추가
			filemap_readahead() : readahead가 설정되어있는 경우 last index로 부터 추가로 data를 읽어옴.(page_cache_async_ra)

f2fs_write_end_io()

```