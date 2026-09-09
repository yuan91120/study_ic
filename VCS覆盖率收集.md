# VCS代码覆盖率收集

## 覆盖率选项

- 使用较多的是

  - -cm
  - -cm_name
  - -cm_dir

  这三个选项**编译**和**仿真**过程都要加。

```
-cm：指定使能覆盖率的类型，包括：line、cond、fsm、tgl、path、branch和assert
-cm_count: 在统计是否覆盖的基础上，进一步统计覆盖的次数
-cm_dir: 指定覆盖率统计结果的存放路径，默认是simv.vdb，更改默认的coverage model生成的目录
-cm_log: 指定编译覆盖率的log文件的名字
-cm obc: 使能可观察覆盖率的编译
-cm_name：修改默认的test目录。对于每一个test，生成的coverage数据，默认是在simv.vdb/snps/coverage/db/testdata/test目录下。比如-cm_name load_test，那么coverage数据，就会生成在simv.vdb/snps/coverage/db/testdata/load_test目录下。
-cm_hier：指定覆盖率统计的范围，可以指定是module名、层次名和源文件等。0表示统计所有，1表示只统计当前层，2表示统计当前层和下一层，之后依次类推。
-cm_tgl mda：为Verilog 2001和SystemVerilog未打包的多维数组启用翻转覆盖
-cm_noconst：告诉VCS不要监视由于信号始终为1或0值而永远无法满足的条件或永远无法执行的线路
-cm_cond：由一个或多个参数指定的修改后的条件覆盖率
-cm_fsmcfg ：指定状态机覆盖率配置文件
-cm_line contassign：收集行覆盖率，并且忽略连续赋值语句
-cm_cond nocasedef：在统计case语句的条件覆盖率时，不考虑default条件未达到的情况

```



编译代码的时候加上以下覆盖率选项：

```
-cm line+cond+fsm+tgl+branch 
-cm_line contassign //收集assign语句的覆盖率
-cm_cond allops+anywidth+event 
-cm_noseqconst 
-debug_all
```



## 覆盖率查看

### 2.1 dve查看

`dve -full64 -cov -dir simv.vdb -elfile uart.el &`



### 2.2 verdi查看

```
verdi -cov -covdir *.vdb/ &
verdi -cov -covdir simv.vdb
```



### 2.3 urg查看

在当前目录下，会生成 urgReport 目录，里面有生成的html文件，使用浏览器即可查看这些文件。

```
urg -dir simv.vdb
firefox urgReport  //表示用浏览器打开覆盖率，进行查看
```



合并覆盖率urg选项

```
urg：Unified Report Generator，将coverage数据转换为html格式
-dir：指定需要拿到的db的hier
-dbname：指定输出的merge db的hier
-elfile：指定exclusive的file，这样更好计算coverage
-elfilelist：忽略中每一个.el文件
-noreport：不输出最终的report，只是merge db
-report：输出report
-format text/both：指定report的输出格式，both意思是两种格式都输出
-matric [line，cond，fsm，tgl，branch，assert]：执行计算的coverage类型
-parallel：并行merge
-full64：以64bit的程序进行merge，如果是64位，也可以不加这个选项
-warn none：忽略warning信息
-metric：+line等等，意思位提取特定的coverage
```



例如：

`urg -full64 -dir *.vdb -dbname merged -parallel -report urgReport`

功能：从当前目录下查找vdb文件，将其覆盖率进行合并，合并后的文件叫做merged.vdb，且是并行merge的，最后输出report，文件名叫做urgReport，里面有html格式的覆盖率。



假如**修改了覆盖率相关的代码**，如新添加了coverpoint等等，

- 希望把新收集的和以前的merge到一起，可以使用选项`-flex_merge union`;
- 不希望合并就使用`-flex_merge drop`

```
urg -full64 -flex_merge union -dbname <merge_coverage_name>.vdb  -dir  simv.vdb &
urg -full64 -flex_merge drop  -dbname <merge_coverage_name>.vdb  -dir  simv.vdb &
```

