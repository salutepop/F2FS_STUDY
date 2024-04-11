# F2FS Write Process

## Outline
1. vfs_write() : virtual file system에서 제공하는 system call
2. f2fs_file_write_iter() : f2fs_node 정보 초기화
3. f2fs_write_begin() : page cache 생성 및 데이터 준비(채우기)
4. page cache write : write back (vs. write through)
5. f2fs_write_end() : page를 최신 상태로 업데이트
6. f2fs_write_data_pages() : writeback 이나 fsync가 trigger될 때, 이 함수를 통해 disk에 쓰기


### f2fs_file_write_iter()

이 함수의 주요 기능은 데이터를 파일에 쓰기 전에 전처리하는 것이며 핵심 프로세스는 파일의 쓰기 위치에 해당하거나 해당하는 값을 초기화하는 것입니다. 예를 들어, 사용자가 4번째 페이지의 위치에 데이터를 작성해야 하는 경우 함수는 먼저 해당 파일을 찾은 다음 4번째 페이지에 해당하는 데이터 블록 주소 레코드를 찾습니다. 이 위치의 값이 `NULL_ADDR` 이면 현재 추가 쓰기(Append Write) 중이므로 값을 `NEW_ADDR`로 초기화하고, 이 위치의 값이 특정 블록 번호이면 덮어쓰기(Overwrite)를 의미하며 별도의 처리가 필요하지 않습니다.

1. file_inode(), iov_iter_count() : 현재 열린 file의 inode를 가져오고, 읽을 수(*bytes*) 를 저장합니다.
2. inode_lock(inode) : inode를 **write lock** 합니다. `down_write(&inode->i_rwsem)`
3. f2fs_write_checks() : write를 위한 무결성을 확인합니다.
4. f2fs_should_use_dio() : flag와 direct i/o(`true`)와 buffered i/o(`false`)를 판단합니다.
5. f2fs_preallocate_blocks() : write할 block을 미리 할당합니다. [관련 commit](https://www.mail-archive.com/linux-f2fs-devel@lists.sourceforge.net/msg20649.html)
   1. inline data가 있고 남아있는 inline data size가 부족하면, f2fs_convert_inline_inode()를 호출합니다. 여기서 inode와 page를 할당하기 위해 `cp_rwsem`을 lock합니다. [Detail](../Lock/f2fs_rwsem.md#f2fs_rwsem-cp_rwsem)
      1. inline data는 3.4kb 이하의 data를 inode block에 직접 쓴 것으로, 추가적인 block을 할당하여 용량이 낭비되는 것을 방지하기 위함.
      2. 만약 새로운 page를 할당받았는데, Data가 있으면 (`f2fs_exist_data()`) data를 storage로 write하고, 해당 inode를 clear함
         1. **`여기는 f2fs_convert_inline_page()의 동작을 컨펌받아야함`**
   2. 이후 `f2fs_map_blocks()`를 통해 연속된 block을 할당합니다.

6. 정상적으로 preallocate가 되면(>=0), [f2fs_dio_write_iter](#f2fs_dio_write_iter) 혹은 [f2fs_buffered_write_iter](#f2fs_buffered_write_iter) 를 호출하여 write를 수행합니다.

7. preallocated blocks 중 미사용 잔여 block을 잘라냄   
   - `f2fs_do_truncate_blocks()` : 여기서 lock 잡음(`f2fs_lock_op(sbi), sbi->cp_rwsem`)
   - fio 평가 trace file에서 *truncate_* 관련 키워드는 호출되지 않아, 유효한 lock이 아닐 것 같음.

8. inode_unlock(inode) : write lock을 해제합니다.
9.  만약 O_DIRECT 인 경우에는, 요청한 i/o offset 만큼 buffer flush를 수행합니다.

### f2fs_dio_write_iter()

### f2fs_buffered_write_iter()
[Source Code @google](https://www.google.com/search?q=bootlin+f2fs_buffered_write_iter())
1. 초기화 과정(file inode 취득, inode로 backing device info 변환)을 진행합니다.
2. `generic_perform_write()`를 통해 iov_iter 수 만큼 write를 반복합니다. (write_begin -> copy_page -> write_end)

### generic_perform_write()
[Source Code @google](https://www.google.com/search?q=bootlin+generic_perform_write()) \
`iov_iter`를 활용하여 Page 단위로 아래의 write과정을 반복합니다.
1. a_ops->write_begin `(f2fs_write_begin)` : page와 연결된 block address를 얻음
   1. f2fs_pagecache_get_page() : page address를 할당받고,
   2. prepare_write_begin() : 해당 페이지와 연결된 node block의 address를 얻음
      1. inline data 인 경우 : `(f2fs_lock_op(sbi))`
      2. offset이 inode size를 초과한 경우 : `(f2fs_down_read(&sbi->node_change))`
      3. f2fs_lookup_read_extent_cache_block() : extent cache의 cached node에서 block address를 가져옴.
         1. 실패하면, `f2fs_get_dnode_of_data()`를 통해 node를 탐색하여 해당 block address를 가져옴.
   3. blk_addr와 node_changed 전달, dnode의 d는 direct/inode 의미

2. copy_page_from_iter_atomic() : page에 data를 복사
3. flush_dcache_page() : D-cache aliasing 문제를 방지하기 위해 copy page 앞뒤로 삽입.
   - 해당 page를 맵핑하는 모든 cache flush [(참고 LINK)](http://barriosstory.blogspot.com/2009/01/flushdcachepage-kmapatomic.html)
  
4. a_ops->write_end `(f2fs_write_end)` : 정상적으로 write 완료되었는지 확인 및 page dirty 및 time 갱신
5. balance_dirty_pages_ratelimited() : system memory의 page 상태 업데이트(프로세스별 dirty page 관리 목적)
   

### f2fs_get_node_info()
NAT에서 node info(index, block address, version)을 얻음
1. cached nat를 찾음 : `f2fs_down_read(&nm_i->nat_tree_lock)`
2. 실패하면, 
   1. f2fs_lookup_journal_in_cursum() : segment summary에서 journal된 nat를 찾음 
      >`down_read(&curseg->journal_rwsem)`
      1. 실패하면, nat_blk으로 node entry 가져옴?\
	  (아래 코드 의미 잘 이해안됨)
			```c
			/* Fill node_info from nat page */
			index = current_nat_addr(sbi, nid);
			f2fs_up_read(&nm_i->nat_tree_lock);

			page = f2fs_get_meta_page(sbi, index);
			if (IS_ERR(page))
				return PTR_ERR(page);

			nat_blk = (struct f2fs_nat_block *)page_address(page);
			ne = nat_blk->entries[nid - start_nid];
			node_info_from_raw_nat(ni, &ne);
			f2fs_put_page(page, 1);
			```
   2. cache_nat_entry() : nat를 nat tree에 caching
      >`f2fs_down_write(&nm_i->nat_tree_lock)`

### f2fs_write_end()
1. PageUptodate(page) : Page(folio)가 최신상태인지 확인, 최신상태가 아니면 page를 최신상태로 바꾸고, 최신상태면 넘어감
2. set_page_dirty() : Page를 dirty 표기
3. unlock page & update time


### f2fs_preallocate_blocks()
write를 위해 write양보다 넉넉한 block을 할당함.\
block할당이 필요없는 경우를 제외하고, map.m_may_create = true 와 flag(`F2FS_GET_BLOCK_PRE_DIO` or `F2FS_GET_BLOCK_PRE_AIO`)를 설정하고 f2fs_map_blocks() 호출하여 block을 할당받음\
문제 없이 할당되면, inode의 flag를 `FI_PREALLOCATED_ALL`로 설정 후 map block의 길이(`map.m_len`) return

### f2fs_map_blocks()
inode와, page offset(write할 시작 점), map.m_len(필요한 사이즈)를 사용하여, 필요한 map blocks의 길이를 할당받아옴 dnode_of_data구조체로
1. struct f2fs_map_blocks를 사용하여 
2. f2fs_get_dnode_of_data() : 

### f2fs_get_dnode_of_data()
file의 write position에 해당하는 data의 block address를 얻는 함수.

### f2fs_build_free_nids()

### f2fs_write_data_pages()
f2fs_write_data_pages() \
   call __f2fs_write_data_pages() \
      call __should_serialize_io() \
      call f2fs_write_cache_pages()

```c
const struct address_space_operations f2fs_dblock_aops = {
	.writepages	= f2fs_write_data_pages,
   ...
}
```

### __f2fs_write_data_pages()
blk_start_plug() ~ blk_finish_plug() [(LINK)](https://lwn.net/Articles/438256/)
- blk_start_plug() : 주어진 blk_plug 구조체를 적절히 초기화한 후에
현재 태스크의 plug 필드에 저장하는데, 만약 blk_start_plug() 함수가 중첩된 실행 경로 상에서
여러 번 호출되었다면 제일 첫 번째로 호출된 경우에만 plug 필드를 저장한다.
이는 plugging 로직이 가장 상위 수준에서 처리될 수 있도록 보장해 준다.
- blk_finish_plug() : 태스크의 plug 필드와 인자로 주어진 blk_plug 구조체가 일치하는 경우에만
동작하며, 대응하는 start 함수와 현재 finish 함수 사이에서 발생한 I/O 연산 (request)들을 모두
I/O 스케줄러에게 전달하고 (insert) 실제로 드라이버가 I/O를 실행하도록 한다.
request를 I/O 스케줄러에게 전달하는 방식은 request의 종류 및 상황에 따라 몇 가지 정책이 적용된다.

```c
	if (__should_serialize_io(inode, wbc)) {
		mutex_lock(&sbi->writepages);
		locked = true;
	}

	blk_start_plug(&plug);
	ret = f2fs_write_cache_pages(mapping, wbc, io_type);
	blk_finish_plug(&plug);

	if (locked)
		mutex_unlock(&sbi->writepages);
```

### f2fs_write_cache_pages()
tag_pages_for_writeback() \
filemap_get_folios_tag() \
f2fs_write_single_data_page() \
f2fs_submit_merged_write_cond()

### tag_pages_for_writeback()
mapping된 pages에서 PAGECACHE_TAG_DIRTY로 설정 된 page를 찾아 PAGECACHE_TAG_TOWRITE로 설정

### filemap_get_folios_tag()
tag (PAGECACHE_TAG_TOWRITE / PAGECACHE_TAG_DIRTY)에 매칭 되는 folio(page)를 찾아 fbatch에 추가하고, 그 갯수(길이)를 return \
fbatch에 추가된 folio는 아래에서 for문 순회하며 f2fs_write_single_data_page()를 통해 write 수행

### f2fs_write_single_data_page()
f2fs_do_write_data_page()를 호출하여 fio의 data를 in-place 및 out-place update 방식으로 write 수행

### f2fs_outplace_write_data()
do_write_page()


### do_write_page()
f2fs_allocate_data_block() \
f2fs_submit_page_write

## Writeback
```
f2fs_write_data_pages()
	if__should_serialize_io() then mutex_lock(&sbi->writepages)
	f2fs_write_cache_pages()
		filemap_get_folios_tag() : write할 folio(page)를 찾아 fbatch에 추가하고, 그 갯수(길이)를 return, fbatch에 추가된 folio는 아래에서 for문 순회하며 f2fs_write_single_data_page()를 통해 write 수행
		for folio in fbatch
			folio_lock(folio)
			f2fs_write_single_data_page()
				f2fs_do_write_data_page()
					"out-place update sequence"
					f2fs_get_dnode_of_data()
						f2fs_get_node_page() - lock_page(page)
					f2fs_trylock_op(fio->sbi)
					f2fs_get_node_info()
						f2fs_down_read(&nm_i->nat_tree_lock)
						__lookup_nat_cache() - spin_lock(&nm_i->nat_list_lock)
						down_read(&curseg->journal_rwsem)
						f2fs_lookup_journal_in_cursum(NAT_JOURNAL)
						up_read(&curseg->journal_rwsem)
						f2fs_up_read(&nm_i->nat_tree_lock)
						f2fs_get_meta_page() - lock_page(page)
						cache_nat_entry()
					set_page_writeback(page) : page cache tagging (dirty/toWrite)
					f2fs_outplace_write_data(&dn, fio)
						f2fs_update_age_extent_cache() - read_lock(&et->lock), write_lock(&et->lock), spin_lock(&eti->extent_lock)
						do_write_page()
							f2fs_allocate_data_block() - f2fs_down_read(&SM_I(sbi)->curseg_lock), mutex_lock(&curseg->curseg_mutex), down_write(&sit_i->sentry_lock), mutex_lock(&dcc->cmd_lock), mutex_lock(&dirty_i->seglist_lock), spin_lock(&io->io_lock)
							f2fs_submit_page_write() : write할 page들을 bio에 채우고, bio 제출 반복 - f2fs_down_write(&io->io_rwsem)
					f2fs_unlock_op(fio->sbi)
				spin_lock(&F2FS_I(inode)->i_size_lock)
				F2FS_I(inode)->last_disk_size = psize
				spin_unlock(&F2FS_I(inode)->i_size_lock)
			folio_unlock(folio)
			
	if__should_serialize_io() then mutex_unlock(&sbi->writepages)

```
