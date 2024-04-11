# super block

### 목적 
Mount된 File system에서 필요한 정보들을 저장합니다.  
첫번째 block (Filesystem Control Block)이 Super block 입니다.

```c
struct super_block 
struct super_operations
```

### super block data
- inode and blocks locations
- file system block size
- maximum filename length
- maximum file size
- the location of the root inode
- file system type


### super operations
- **alloc_inode** : struct <fsname>_inode_info 구조를 할당하고 기본 VFS inode 초기화를 수행합니다.
  -  alloc_inode() 함수는 new_inode() 및 iget_locked() 함수에 의해 호출됩니다.
- **write_inode** : 매개변수로 받은 inode를 디스크에 저장/업데이트합니다.
	- sb_bread() 함수를 사용하여 디스크에서 inode를 로드합니다.
	- 저장된 inode에 따라 버퍼를 수정합니다.
	- mark_buffer_dirty()를 사용하여 버퍼를 더티로 표시합니다. 그러면 커널이 디스크 쓰기를 처리합니다.
- **evict inode** : 디스크 및 메모리(디스크의 inode 및 관련 데이터 블록 모두)에서 i_ino 필드에 수신된 번호가 있는 inode에 대한 정보를 제거합니다.
	- 디스크에서 inode 삭제합니다.
	- 디스크 비트맵 업데이트(있는 경우) 합니다.
	- truncate_inode_pages()를 호출하여 페이지 캐시에서 inode를 삭제합니다.
	- delete the inode from memory by calling clear_inode()
- **destroy_inode** : inode가 차지하는 메모리를 해제합니다.

### Initialize
fill_super() 함수가 호출되어 수퍼블록 초기화가 완료되며, 초기화 과정에는 struct super_block 구조 필드 채우기 및 루트 디렉토리 inode 초기화가 포함됩니다.  
`umount`과정에서 put_super()를 통해 Superblock이 해제됩니다.