# Filebench 비교

### Webproxy, Personal PC

**`Thread 8`**

```
<EXT4>
closefile1           4010865ops    66839ops/s   0.0mb/s    0.001ms/op [0.000ms - 0.749ms]
createfile1          4010866ops    66839ops/s   0.0mb/s    0.057ms/op [0.009ms - 16.408ms]
deletefile1          4010871ops    66840ops/s   0.0mb/s    0.058ms/op [0.008ms - 16.166ms]
61.262: IO Summary: 12032602 ops 200518.355 ops/s 0/0 rd/wr   0.0mb/s 0.039ms/op

<F2FS>
closefile1           1839466ops    30654ops/s   0.0mb/s    0.001ms/op [0.000ms - 0.571ms]
createfile1          1839467ops    30654ops/s   0.0mb/s    0.130ms/op [0.012ms - 63.307ms]
deletefile1          1839470ops    30654ops/s   0.0mb/s    0.126ms/op [0.008ms - 63.174ms]
61.252: IO Summary: 5518403 ops 91962.651 ops/s 0/0 rd/wr   0.0mb/s 0.086ms/op
```

**`Thread 8 + ftrace Profile`**

```
<EXT4>
closefile1           1377187ops    22951ops/s   0.0mb/s    0.001ms/op [0.000ms - 0.343ms]
createfile1          1377187ops    22951ops/s   0.0mb/s    0.168ms/op [0.016ms - 16.189ms]
deletefile1          1377191ops    22951ops/s   0.0mb/s    0.176ms/op [0.016ms - 12.390ms]
61.597: IO Summary: 4131565 ops 68853.086 ops/s 0/0 rd/wr   0.0mb/s 0.115ms/op


<F2FS>
closefile1           1455455ops    24255ops/s   0.0mb/s    0.001ms/op [0.001ms - 0.653ms]
createfile1          1455455ops    24255ops/s   0.0mb/s    0.166ms/op [0.022ms - 79.412ms]
deletefile1          1455461ops    24255ops/s   0.0mb/s    0.159ms/op [0.012ms - 75.347ms]
61.322: IO Summary: 4366371 ops 72764.389 ops/s 0/0 rd/wr   0.0mb/s 0.109ms/op
```
### Createfiles 1M, Personal PC
**`Thread 8`**
```
<EXT4>
closefile1           999994ops    90896ops/s   0.0mb/s    0.001ms/op [0.001ms - 1.789ms]
createfile1          1000000ops    90897ops/s   0.0mb/s    0.082ms/op [0.012ms - 8.030ms]
13.722: IO Summary: 1999994 ops 181793.210 ops/s 0/0 rd/wr   0.0mb/s 0.042ms/op

<F2FS>
closefile1           999993ops    55549ops/s   0.0mb/s    0.001ms/op [0.001ms - 3.406ms]
createfile1          1000000ops    55549ops/s   0.0mb/s    0.134ms/op [0.012ms - 412.155ms]
20.631: IO Summary: 1999993 ops 111097.835 ops/s 0/0 rd/wr   0.0mb/s 0.068ms/op
```
**`Thread 8 + semi-ftrace Profile`**
```
<EXT4>
closefile1           999993ops    90900ops/s   0.0mb/s    0.001ms/op [0.001ms - 2.473ms]
createfile1          1000000ops    90900ops/s   0.0mb/s    0.085ms/op [0.011ms - 8.130ms]
13.732: IO Summary: 1999993 ops 181800.076 ops/s 0/0 rd/wr   0.0mb/s 0.043ms/op

<F2FS>
closefile1           999993ops    58816ops/s   0.0mb/s    0.001ms/op [0.000ms - 1.351ms]
createfile1          1000000ops    58817ops/s   0.0mb/s    0.130ms/op [0.013ms - 412.334ms]
19.617: IO Summary: 1999993 ops 117633.375 ops/s 0/0 rd/wr   0.0mb/s 0.066ms/op
```


**`Thread 8 + ftrace Profile`**

| Function            | Hit     | Time(ms)  | Avg(us) | s^2(us)   | time_ratio(%) |
|---------------------|---------|-----------|---------|-----------|---------------|
| ext4_create         | 1000001 |  8966.082 |   9.121 |    33.123 |          50.3 |
| __ext4_new_inode    | 1010126 |  6137.845 |   6.207 |    28.578 |          34.4 |
| ext4_add_nondir     | 1000001 |   2502.97 |   2.535 |     3.179 |          14.0 |
| __ext4_journal_stop | 1024651 |   217.484 |   0.214 |     0.025 |           1.2 |

| Function            | Hit     | Time(ms)  | Avg(us) | s^2(us)   | time_ratio(%) |
|---------------------|---------|-----------|---------|-----------|---------------|
| f2fs_create         | 1000000 | 14720.847 |  14.004 | 17532.831 |          51.2 |
| f2fs_do_add_link    | 1010125 |  5007.473 |   4.985 |     9.471 |          17.4 |
| f2fs_balance_fs     | 1010125 |  4687.434 |   3.906 |  17549.96 |          16.3 |
| f2fs_new_inode      | 1010125 |  4356.364 |   4.353 |    92.674 |          15.1 |
```
<EXT4>
closefile1           999993ops    32254ops/s   0.0mb/s    0.001ms/op [0.001ms - 4.032ms]
createfile1          1000000ops    32254ops/s   0.0mb/s    0.241ms/op [0.018ms - 9.099ms]
34.063: IO Summary: 1999993 ops 64508.327 ops/s 0/0 rd/wr   0.0mb/s 0.121ms/op

<F2FS>
closefile1           999993ops    47614ops/s   0.0mb/s    0.001ms/op [0.001ms - 0.258ms]
createfile1          1000000ops    47614ops/s   0.0mb/s    0.165ms/op [0.016ms - 513.659ms]
23.767: IO Summary: 1999993 ops 95227.777 ops/s 0/0 rd/wr   0.0mb/s 0.083ms/op
```



## Createfiles 100k, Hexa
**`Thread 8`**
```
<EXT4>
closefile1           99993ops     7139ops/s   0.0mb/s    0.011ms/op [0.009ms - 0.267ms]
createfile1          100000ops     7140ops/s   0.0mb/s    1.060ms/op [0.152ms - 16.191ms]
15.695: IO Summary: 199993 ops 14278.578 ops/s 0/0 rd/wr   0.0mb/s 0.536ms/op

<F2FS>
closefile1           99993ops     6248ops/s   0.0mb/s    0.011ms/op [0.009ms - 4.068ms]
createfile1          100000ops     6248ops/s   0.0mb/s    1.232ms/op [0.160ms - 4731.869ms]
17.499: IO Summary: 199993 ops 12496.152 ops/s 0/0 rd/wr   0.0mb/s 0.621ms/op



```

webproxy
1. block trace draw
2. workload 세분화 후 비교
   - create - delete 반복
   - close - open 반복   
   - open - read - close 반복
3. open/close에서부터 차이가 발생할 것 같은데, 그렇다하면 file open과정 찾아보기
  - inode lock? 이면..흠
4. 만약 release 차이이면 개선할 방법 있는지 고민
   - f2fs : close == release 수행
   - ext4 : close != release, all of files close == release 수행

struct f2fs_inode_info {

        unsigned long sum_filetime[F_MAX];      /* F_OPEN, F_CLOSE */
        unsigned long sum_filecount[F_MAX];     /* F_OPEN, F_CLOSE */
};

        struct f2fs_inode_info *fi = F2FS_I(inode);

unsigned int wait_ms = 0;
wait_ms = jiffies_to_msecs(delta);   

<EXT4>
closefile1           3447984ops    57462ops/s   0.0mb/s    0.001ms/op [0.000ms - 0.593ms]
createfile1          3447984ops    57462ops/s   0.0mb/s    0.136ms/op [0.009ms - 24.066ms]
deletefile1          3447993ops    57462ops/s   0.0mb/s    0.137ms/op [0.008ms - 24.097ms]
61.332: IO Summary: 10343961 ops 172386.476 ops/s 0/0 rd/wr   0.0mb/s 0.091ms/op

7.749: Per-Operation Breakdown
closefile1           499986ops    83323ops/s   0.0mb/s    0.001ms/op [0.001ms - 2.341ms]
createfile1          500000ops    83325ops/s   0.0mb/s    0.186ms/op [0.012ms - 7.723ms]
7.749: IO Summary: 999986 ops 166648.391 ops/s 0/0 rd/wr   0.0mb/s 0.094ms/op

<F2FS>
closefile1           1646208ops    27435ops/s   0.0mb/s    0.001ms/op [0.000ms - 0.600ms]
createfile1          1646209ops    27435ops/s   0.0mb/s    0.291ms/op [0.016ms - 67.303ms]
deletefile1          1646215ops    27435ops/s   0.0mb/s    0.287ms/op [0.008ms - 67.200ms]
61.253: IO Summary: 4938632 ops 82304.516 ops/s 0/0 rd/wr   0.0mb/s 0.193ms/op


11.218: Per-Operation Breakdown
closefile1           499985ops    55547ops/s   0.0mb/s    0.001ms/op [0.001ms - 0.205ms]
createfile1          500000ops    55549ops/s   0.0mb/s    0.271ms/op [0.013ms - 215.848ms]
11.218: IO Summary: 999985 ops 111096.027 ops/s 0/0 rd/wr   0.0mb/s 0.136ms/op


-------------
hexa3

<EXT4>  th4 w/o profile
closefile1           781805ops    13029ops/s   0.0mb/s    0.002ms/op [0.001ms - 0.380ms]
createfile1          781805ops    13029ops/s   0.0mb/s    0.143ms/op [0.034ms - 8.755ms]
deletefile1          781807ops    13029ops/s   0.0mb/s    0.157ms/op [0.033ms - 8.601ms]
62.023: IO Summary: 2345417 ops 39086.480 ops/s 0/0 rd/wr   0.0mb/s 0.100ms/op


<F2FS> th4 w/o profile
closefile1           981358ops    16355ops/s   0.0mb/s    0.002ms/op [0.001ms - 0.549ms]
createfile1          981358ops    16355ops/s   0.0mb/s    0.123ms/op [0.036ms - 147.815ms]
deletefile1          981361ops    16355ops/s   0.0mb/s    0.114ms/op [0.022ms - 148.073ms]
61.498: IO Summary: 2944077 ops 49063.625 ops/s 0/0 rd/wr   0.0mb/s 0.080ms/op


-----------------
personal - 231122

<EXT4>
closefile1           3462819ops    57709ops/s   0.0mb/s    0.001ms/op [0.000ms - 3.922ms]
createfile1          3462821ops    57709ops/s   0.0mb/s    0.136ms/op [0.009ms - 16.140ms]
deletefile1          3462828ops    57709ops/s   0.0mb/s    0.136ms/op [0.008ms - 20.208ms]
61.258: IO Summary: 10388468 ops 173127.823 ops/s 0/0 rd/wr   0.0mb/s 0.091ms/op


closefile1           1193439ops    19889ops/s   0.0mb/s    0.001ms/op [0.001ms - 0.173ms]
createfile1          1193439ops    19889ops/s   0.0mb/s    0.396ms/op [0.016ms - 15.563ms]
deletefile1          1193445ops    19889ops/s   0.0mb/s    0.403ms/op [0.017ms - 15.654ms]
61.604: IO Summary: 3580323 ops 59667.809 ops/s 0/0 rd/wr   0.0mb/s 0.267ms/op

<F2FS>
closefile1           1301242ops    21686ops/s   0.0mb/s    0.001ms/op [0.001ms - 0.485ms]
createfile1          1301242ops    21686ops/s   0.0mb/s    0.369ms/op [0.022ms - 75.866ms]
deletefile1          1301250ops    21686ops/s   0.0mb/s    0.363ms/op [0.012ms - 75.668ms]
61.313: IO Summary: 3903734 ops 65056.960 ops/s 0/0 rd/wr   0.0mb/s 0.244ms/op


---------------------
personal - th8

<EXT4> with profile
closefile1           1352332ops    22537ops/s   0.0mb/s    0.001ms/op [0.000ms - 0.198ms]
createfile1          1352333ops    22537ops/s   0.0mb/s    0.171ms/op [0.016ms - 12.310ms]
deletefile1          1352336ops    22537ops/s   0.0mb/s    0.179ms/op [0.016ms - 12.270ms]
61.604: IO Summary: 4057001 ops 67611.591 ops/s 0/0 rd/wr   0.0mb/s 0.117ms/op


<F2FS> with profile
closefile1           1456654ops    24275ops/s   0.0mb/s    0.001ms/op [0.001ms - 4.010ms]
createfile1          1456655ops    24275ops/s   0.0mb/s    0.165ms/op [0.018ms - 81.121ms]
deletefile1          1456659ops    24275ops/s   0.0mb/s    0.160ms/op [0.011ms - 77.252ms]
61.319: IO Summary: 4369968 ops 72824.924 ops/s 0/0 rd/wr   0.0mb/s 0.109ms/op



<EXT4> without profile
closefile1           4010865ops    66839ops/s   0.0mb/s    0.001ms/op [0.000ms - 0.749ms]
createfile1          4010866ops    66839ops/s   0.0mb/s    0.057ms/op [0.009ms - 16.408ms]
deletefile1          4010871ops    66840ops/s   0.0mb/s    0.058ms/op [0.008ms - 16.166ms]
61.262: IO Summary: 12032602 ops 200518.355 ops/s 0/0 rd/wr   0.0mb/s 0.039ms/op

<F2FS>without profile
closefile1           1839466ops    30654ops/s   0.0mb/s    0.001ms/op [0.000ms - 0.571ms]
createfile1          1839467ops    30654ops/s   0.0mb/s    0.130ms/op [0.012ms - 63.307ms]
deletefile1          1839470ops    30654ops/s   0.0mb/s    0.126ms/op [0.008ms - 63.174ms]
61.252: IO Summary: 5518403 ops 91962.651 ops/s 0/0 rd/wr   0.0mb/s 0.086ms/op