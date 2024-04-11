1. f2fs_write_checkpoint
SATA / PM 의 f2fs_write_checkpoint 수행 시간 분석 (th1, 단위 usec)
* 10만개 파일 생성 시점인 1st 결과
    : block ops는 2배, NAT/SIT entries는 3배 차이남

* 나머지 결과
    : block ops는 동등, NAT/SIT entries는 5배 차이남
* 결론
  * WRITE 성능에 2배, READ성능에 3-5배 영향 받음
  * WRITE는 MERGE하고, READ는 RA를 활용해보자

1st (us)        SATA	  PM	
block_ops	      956059	485933	// 4-128KB Write
merged writes	  4   	  2	
nat/sit entries	92813	  30501	  // 4KB Read (LBA 연속)
do CP	          10464	  941	    // 4-128KB Write
 
etc	(나머지)    SATA	  PM	
block_ops	      9998	  10285	  // 4-128KB Write
merged writes	  0	      1	
nat/sit entries	33125	  6867	  // 4KB Read (LBA 연속)
do CP	          7544	  75	    // 4-128KB Write

2. block_operations
왜 quota 시간이 오래 걸리는지 파악 (시간을 더하는 방식으로 계산하기)
 > 지금 방식으로 계산하면 retry로 인해서 시간이 정상적이지 않음..(이전 데이터들도 다 문제 있음)

 측정 결과 : 
th1 / pm
blk_op calls : 114
blk_op time QUOTA : 51781 ns (cnt: 340)
blk_op time DENTS : 670458494 ns (cnt: 340)
blk_op time NODES : 928178670 ns (cnt: 227)

th1 / sata
blk_op calls : 64
blk_op time QUOTA : 26364 ns (cnt: 190)
blk_op time DENTS : 440051023 ns (cnt: 190)
blk_op time NODES : 1154289834 ns (cnt: 128)

f2fs_write_checkpoint 함수를 숨기면 될듯함,
위 함수 skip한 create 10 sec 평가 결과
<기존>
sata : 286081 ops
pm :   443142 ops
<skip>
sata : 469602 ops
pm :   465771 ops

block ops : quota 및 dirty dentry page 


delay 한번에
skip
