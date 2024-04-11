# file

### 목적
물리적인 Disk와 무관하게 VFS entity로만 존재하며, user와 가장 가까운 File system 구성요소입니다.  
Inode는 Filesystem 구현 관점에서 disk의 file을 추상화하고, File은 Process 관점에서 open file을 추상화 하는 차이가 있음. 

### file data
- file cursor position
- file opening rights
- pointer to the associated inode (eventually its index)