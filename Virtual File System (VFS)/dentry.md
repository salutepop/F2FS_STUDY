# dentry (Directory entry)

### 목적
주요 작업은 `inode`와 `filename`을 연결하는 것입니다.
- an integer that identifies the inode
- a string representing its name.

```c
struct dentry
struct dentry_operations
```

### dentry data
- **d_inode** : dentry가 참조하는 inode
- **d_parent** : 상위 디렉토리와 관련된 dentry;
- **d_name** : 필드 name 및 len(이름 및 이름 길이)을 포함하는 struct qstr 구조체
- **d_op** : 커널은 기본 작업을 구현하므로 이를 (재)구현할 필요가 없으나, 일부 파일 시스템은 dentry의 특정 구조를 기반으로 최적화를 수행할 수 있습니다.
- **d_fsdata** : dentry 작업을 구현하는 파일 시스템용으로 예약된 필드입니다.

### dentry operations
- **d_make_root**: 루트 dentry 할당, 일반적으로 루트 디렉터리를 초기화해야 하는 슈퍼 블록(fill_super)을 읽기 위해 호출되는 함수에서 사용됩니다. super_block 구조체의 s_root field를 채우는 함수
- **d_add** : dentry를 inode와 연결합니다. 위에서 설명한 호출에서 매개변수로 수신된 dentry는 생성해야 하는 항목(이름, 길이)을 나타냅니다. 이 함수는 관련된 dentry가 없고 아직 inode의 해시 테이블에 서 찾을 수 없는 새 inode를 생성/로드할 때 사용됩니다.
- **d_instantiate** :  사전에 dentry가 해시 테이블에 추가된 이전 호출의 더 가벼운 버전입니다. d_instantiate는 생성 호출(mkdir, mknod, rename, symlink) 및 NOT d_add를 구현하는 데 사용해야 합니다.