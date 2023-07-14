# inode

### 목적
File에 대한 모든 정보를 함축하고 있으며, 일반적으로 Data block과 문기뢴 Special area로 그룹화 되어있습니다.  
일반적으로 `Filename`은 `inode`가 아닌 `dentry entity`에 저장되어 있습니다. (multiple hard links)

```c
struct inode 
struct inode_operations
```

### inode data
- file type
- file size
- access rights
- access or modify time
- location of data on the disk (pointers to disk blocks containing data).

### inode get
- **f2fs_iget()** :  VFS inode가 존재하는 경우 이를 찾거나 새 것을 생성하고 디스크의 정보로 채우는 역할을 합니다.
- **iget_locked()** : f2fs_iget()이 호출하는 함수로 VFS에서 inode 구조를 가져옵니다
- inode가 새로 생성되면 디스크에서 inode를 읽고(sb_bread() 사용) 유용한 정보를 채워야 합니다.
  
### inode operations
- **setattr** : 붙여넣기 크기가 inode의 현재 크기와 다른 경우 잘라내기 작업을 수행해야 합니다. 
- **truncate operation**
	- 현재 여분인 디스크의 데이터 블록 해제(새 차원이 이전 차원보다 작은 경우) 또는 새 블록 할당(새 차원이 더 큰 경우)
	- updating disk bit maps (if used)
	- updating the inode;
    - block_truncate_page() 함수를 사용하여 마지막 블록에서 사용되지 않은 공간을 0으로 채웁니다.