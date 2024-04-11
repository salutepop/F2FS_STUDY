- create 과정에서 set_page_dirty() 시간 측정
  - dirty_filio
- create 과정에서 grab_cache_page() 시간 측정
  - f2fs_grab_cache_page inline 함수 안에서 sbi에 기록

#include "linux/ktime.h"
signed long long (s64)
ktime_get_raw(void) //ns

실제 소요 시간 20초
dirtyTime META : 2648038
dirtyTime NODE : 936257210
dirtyTime DATA : 14623007

dirtyTime META : 2644410
dirtyTime NODE : 950701570
dirtyTime DATA : 14619727


```
ftrace
node: 1.46초
data: 0.026초
meta : 0.004초
```
closefile1           999993ops    55550ops/s   0.0mb/s    0.001ms/op [0.000ms - 0.214ms]     
createfile1          1000000ops    55550ops/s   0.0mb/s    0.137ms/op [0.013ms - 428.674ms]  


- create 과정에서 set_page_dirty() 시간 측정
- create 과정에서 grab_cache_page() 시간 측정

f2fs_dirty_meta_folio,
f2fs_dirty_data_folio,
f2fs_dirty_node_folio,

signed long long pagetime_meta;
signed long long pagetime_node;
signed long long pagetime_data;


s64 ktime_before = ktime_get_raw();
pagetime_node += (ktime_get_raw() - ktime_before);

dirtyTime META : 4074020
dirtyTime NODE : 1353206400
dirtyTime DATA : 59881276

closefile1           999993ops    58817ops/s   0.0mb/s    0.001ms/op [0.000ms - 2.346ms]
createfile1          1000000ops    58817ops/s   0.0mb/s    0.131ms/op [0.013ms - 17.500ms]
38.888: IO Summary: 1999993 ops 117633.583 ops/s 0/0 rd/wr   0.0mb/s 0.066ms/op

struct f2fs_sb_info *sbi = F2FS_M_SB(mapping)


단위 (초)		
	                th4	            th8	            th16
f2fs_create	        14.53107177	    16.3583701	    18.1100315
f2fs_balance_fs	    4.741817214	    5.968332049	    5.2665038
f2fs_do_add_link	5.197058941	    6.05132252	    6.9253081
f2fs_new_inode	    3.973917755 	4.58225378	    5.0392673
f2fs_alloc_nid_done	0.266036068	    0.314800973	    0.33944213
			
set_page_dirty META	0.002486453 	0.002655662	    0.002555493
set_page_dirty NODE	0.828737877 	0.955664006	    1.051805948
set_page_dirty DATA	0.013150014 	0.014763829	    0.016739902
grab_cache_page	    2.231920319	    2.49985579	    2.732807508



1) page cache 없애기
grab page cache -> alloc page / alloc_pages 해보기
direct write 하는 것처럼 direct create option 추가 (page cache 안쓰는)

grab_cache_page 호출 함수들
f2fs_new_node_page
__get_node_page (3M 회 호출 됨)
f2fs_update_inode_page
f2fs_get_dnode_of_data
f2fs_add_inline_entry

2) check point 단축
  max 성능 보기 위해서  node.h //  #define DEF_NAT_CACHE_THRESHOLD         100000
   1) check point 시간 줄이기
   2) check point 없애기
      1) nid 만을 위한 가벼운 consistency기법 고안


