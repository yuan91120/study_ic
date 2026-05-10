- [1. 覆盖率的作用](#1-覆盖率的作用)
- [2. 覆盖率类型](#2-覆盖率类型)
  - [代码覆盖率](#代码覆盖率)
  - [功能覆盖率](#功能覆盖率)
  - [断言覆盖率](#断言覆盖率)
- [3. 覆盖组](#3-覆盖组)
  - [类中定义覆盖率](#类中定义覆盖率)
  - [覆盖组的触发](#覆盖组的触发)
- [4. 数据采样](#4-数据采样)
  - [自定义仓](#自定义仓)
  - [条件覆盖率](#条件覆盖率)
  - [为枚举类型创建仓](#为枚举类型创建仓)
  - [翻转覆盖率](#翻转覆盖率)
  - [通配符`wildcard`](#通配符wildcard)
  - [忽略值`ignore_bins`](#忽略值ignore_bins)
  - [不合法的仓`illegal_bins`](#不合法的仓illegal_bins)
- [5. 交叉覆盖率](#5-交叉覆盖率)
  - [对交叉覆盖仓进行标号](#对交叉覆盖仓进行标号)
  - [排除部分交叉覆盖仓](#排除部分交叉覆盖仓)
  - [覆盖率权重`weight`](#覆盖率权重weight)
  - [使用仓名的交叉覆盖率](#使用仓名的交叉覆盖率)
  - [使用binsof的交叉覆盖率](#使用binsof的交叉覆盖率)
- [6. 覆盖组传参数](#6-覆盖组传参数)
  - [数值传递](#数值传递)
  - [引用传递](#引用传递)
- [7. 覆盖选项](#7-覆盖选项)

# 1. 覆盖率的作用

- 衡量芯片功能验证是否完备的一种指标
- 实现方式：
  - 受约束的随机化测试
  - 定向测试



# 2. 覆盖率类型

## 代码覆盖率

- 衡量设计中代码的执行情况

- 类型包括

  - 行覆盖率

    每行代码是否被执行过

  - 分支覆盖率

    `if-else`或`case`语句中每个分支是否都走过

  - 条件覆盖率

    

  - 翻转覆盖率

    每个信号是否都经过`0->1`和`1->0`的跳变

  - 状态机覆盖率

    每个状态是否都达到过；每个跳转路径是否都经历过

- 代码覆盖率达到100%，并不代表功能完整



- 目标：90-95%

  会有一些冗余/调试代码

## 功能覆盖率

- 设计的功能点是否达标



- 目标：力求100%，通常公司要求95-98%。实际可以达到100%



## 断言覆盖率

- 检验设计中的两个内部信号之间关系的代码



# 3. 覆盖组

- 可以定义在**类**中，也可定义在**程序或模块**层次；
- 包含覆盖点`coverpoint`、选项`option`、形式参数`argument`和可选触发`trigger event`；
- 一次定义后，可多次实例化
- 覆盖组需要实例化之后才能进行采样



- 覆盖组定义

```verilog
class Trans;
    rand bit[31:0] data;
    rand bit[2:0] port;
endclass

covergroup CovPort;
    coverpoint tr.port; //覆盖点
endgroup

initial begin
    Trans tr;
    CovPort ck;
    ck=new(); // 覆盖组实例化
    tr=new();
    repeat(32)begin
        assert(tr.randomize);
        ifc.cb.port<=tr.port;
        ifc.cb.data<=tr.data;
        ck.sample(); //覆盖组采样
        @ifc.cb;
    end
end
```





## 类中定义覆盖率

- 类中的功能覆盖率

```verilog
class Trans;
    Trans tr;
    mailbox mbx_in;
    covergroup CovPort;
        coverpoint tr.port;
    endgroup
    
    function new(mailbox mbx_in);
        CovPort=new();
        this.mbx_in=mbx_in;
    endfunction
    
    task main;
        forever begin
            tr=mbx_in.get;
            ifc.cb.port<=tr.port;
            ifc.cb.data<=tr.data;
            CovPort.sample(); //收集覆盖率
        end
    endtask
endclass
```



## 覆盖组的触发

功能覆盖率主要关注两个部分：

- 采样的数据
- 采样数据的时刻



触发采样的方式：

- sample函数
- 在covergroup的定义中采用阻塞表达式（wait或@实现信号或事件的阻塞）



- 使用回调函数进行采样
- 使用事件触发采样
- 使用SV断言触发采样



# 4. 数据采样

仓bin：记录每个数值被捕捉的次数，衡量功能覆盖率的基本单位

域：所有可能数值的个数

覆盖率：采样值的数目除以域中仓的数目



- 限制自动创建bin的数目：

  `auto_bin_max`指明了自动**创建bin的最大数目，缺省值为64**；

  例如：16bits变量有65536个可能的值，SV会把这些值平均分配到64个仓中，每个仓覆盖1024个值；

  限制仓的最大数目：

  ```verilog
  // 作为一个覆盖点的选项使用
  covergroup CovPort;
      coverpoint tr.port {options.auto_bin_max=2;} //分成2个仓
  endgroup
  
  // 作为整个覆盖组的选项使用
  covergroup CovPort;
      options.auto_bin_max=2; //分成2个仓 同时影响port data
      coverpoint tr.port；
      coverpoint tr.data;
  endgroup
  ```



## 自定义仓

- 自动生成的仓：适用于匿名数值，如计数值、地址值或2的幂值。

- SV会自动为枚举类型的仓命名，其他变量需要自行命名
- 命名仓的方式：`[]`
- 自定义仓之后，SV不再自动创建仓，计算覆盖率时只会使用自定义的仓

```verilog
covergroup CovLen;
    len:coverpoint (tr.hdr_len+tr.payload_len+5'b0) {
        bins len[]={0:23};
    }
endgroup
```



- 自定义仓的漏洞

  最长的头`hdr_len`（3bits）：7

  最长的负载`payload_len`（4bits）：15

  总共长度：22



- 4bits变量kind进行采样——自定义仓

  ```verilog
  covergroup CovKind;
      coverpoint tr.kind{
          bins zero={0}; // 1个仓 kind==0
          bins lo={[1:3],5}; //1个仓 代表1：3和5
          bins hi[]={[8:$]}; //8个独立的仓：8...15
          bins misc=default; //1个仓 代表剩余的值
      }
  endgroup
  ```

  - default的使用

  - `$`的使用：编译器自动计算范围的边界

    ```verilog
    int i;
    covergroup range_cover;
        coverpoint i{
            bins neg={[$:-1]}; //负值 32'h800_0000 : -1
            bins zero={0}; //零值
            bins pos={[1:$]}; //正值 1：32'h7FFF_FFFF
        }
    endgroup
    ```

    

## 条件覆盖率

- 关键字：`iff`

最常使用情况：复位期间关闭覆盖

```verilog
// 复位期间禁止
covergroup CoverPort;
    coverpoint port iff(!bus_if.reset); //reset==1时不收集覆盖率数据
endgroup
```



- start和stop函数的使用

```verilog
initial begin
    CovPort ck=new(); //实例化覆盖组
    #1ns ck.stop();
    bus.if.reset=1;
    #100ns bus_if.reset=0;
    ck.start();
    ...
end
```

  

## 为枚举类型创建仓

对于枚举类型，SV会为每个可能的值创建一个仓

```verilog
typedef enum{INIT,DECODE,IDLE} fsm_e;
fsm_e pstate,nstate;
covergroup cg_fsm;
    coverpoint pstate;
endgroup 
```



## 翻转覆盖率

- 确定覆盖点翻转次数

```verilog
covergroup CovPort;
    coverpoint port{
        bins t1=(0=>1),(0=>2),(0=>3);
    }
endgroup
```



- 缩略形式（0=>1=>1=>1=>2）

  `form:(0=>1[*3]=>2)`

  对数值1进行3/4/5次重复：`1[*3:5]`



## 通配符`wildcard`

关键词：wildward

在表达式中，任何X,Z或?都会被当成0或1的通配符

```verilog
bit[2:0]port;
covergroup CovPort;
    coverpoint port{
        wildcard bins even={3'b??0};
        wildcard bins odd={3'b??1};
    }
endgroup
```



## 忽略值`ignore_bins`

3bits变量只用来存0~5共6个值时，需要忽略其他值，再进行覆盖率计算

```verilog
bit[2:0] low_ports_0_5;
covergroup CovPort;
    coverpoint low_ports_0_5{
        ignore_bins hi={[6,7]}; //忽略最后两个仓
    }
endgroup
```



## 不合法的仓`illegal_bins`

有些采样值不仅需要被忽略，还需要报错

```verilog
bit[2:0] low_ports_0_5;
covergroup CovPort;
    coverpoint low_ports_0_5{
        illegal_bins hi={[6,7]}; //如果出现报错
    }
endgroup
```



# 5. 交叉覆盖率

关键词：`cross`

- 基本交叉覆盖率的例子

  SV会创建16*8=126个仓

```verilog
class Trans;
    rand bit[3:0] kind;
    rand bit[2:0] port;
endclass
Trans tr;
covergroup CovPort;
    kind:coverpoint tr.kind;
    port:coverpoint tr.port;
    cross kind,port; //将kind和port交叉
endgroup
```



## 对交叉覆盖仓进行标号

- 指定交叉覆盖仓的名称，

  交叉覆盖仓数变为：8*11=88个

```verilog
covergroup CovPort;
    port:coverpoint tr.port{
        bins port[]={[0:$]};
    }
    kind:coverpoint tr.kind{
        bins zero={0}; // 1个仓 kind==0
        bins lo={[1:3],5}; //1个仓 代表1：3和5
        bins hi[]={[8:$]}; //8个独立的仓：8...15
        bins misc=default; //1个仓 代表剩余的值
    }
    cross kind,port; //将kind和port交叉
endgroup
```



## 排除部分交叉覆盖仓

关键词：`ignore_bins`

指定覆盖点：`binsof`

指定数值集：`intersect`

```verilog
covergroup Covport;
    port:coverpoint tr.port{
        bins port[]={[0:$]};
    }
    kind:coverpoint tr.kind{
        bins zero={0}; // 1个仓 kind==0
        bins lo={[1:3],5}; //1个仓 代表1：3和5
        bins hi[]={[8:$]}; //8个独立的仓：8...15
        bins misc=default; //1个仓 代表剩余的值
    }
    cross kind,port{
        ignore_bins hi=binsof(port)intersect{7};
        ignore_bins md=binsof(port)intersect{0}&&
        			   binsof(kind)intersect{[9:11]};
        ignore_bins lo=binsof(kind.lo);
    }
endgroup
```



## 覆盖率权重`weight`

关键词：`option.weight`

```verilog
covergroup Covport;
    port:coverpoint tr.port{
        bins port[]={[0:$]};
        option.weight=0; //在总体中不占任何分量
    }
    kind:coverpoint tr.kind{
        bins zero={0}; // 1个仓 kind==0
        bins lo={[1:3],5}; //1个仓 代表1：3和5
        bins hi[]={[8:$]}; //8个独立的仓：8...15
        bins misc=default; //1个仓 代表剩余的值
        option.weight=5; //在总体中所占分量
    }
    cross kind,port{
        option.weight=10; //给予交叉更高权重
    }
endgroup
```



## 使用仓名的交叉覆盖率

对于两个随机变量a和b，只对三种状态感兴趣：

- {a\==0, b==0}
- {a\==1, b==0}
- {b==1}

```verilog
class Trans;
    rand bit a,b;
endclass
Trans tr;
covergroup CrossBinNames;
    a:coverpoint tr.a{
        bins a0={0}；
        bins a1={1};
        option.weight=0;
    }
    b:coverpoint tr.b{
        bins b0={0}；
        bins b1={1};
        option.weight=0;
    }
    ab:cross a,b{
        bins a0b0=binsof(a.a0)&&binsof(b.b0);
        bins a1b0=binsof(a.a1)&&binsof(b.b0);
        bins b1=binsof(b.b1);
    }
endgroup
```



## 使用binsof的交叉覆盖率

```velocity
class Trans;
    rand bit a,b;
endclass
Trans tr;
covergroup CrossBinNames;
    a:coverpoint tr.a{
        option.weight=0;
    }
    b:coverpoint tr.b{
        option.weight=0;
    }
    ab:cross a,b{
        bins a0b0=binsof(a)intersect{0}&&binsof(b)intersect{0};
        bins a1b0=binsof(a)intersect{1}&&binsof(b)intersect{0};
        bins b1=binsof(b)intersect{1};
    }
endgroup
```



# 6. 覆盖组传参数

## 数值传递

```verilog
bit [2:0] port;
covergroup CovPort(int mid);
    coverpoint port{
        bins lo={[0:mid-1]};
        bins hi={[mid:$]};
    }
endgroup
CovPort cp;
initial begin
    cp=new(5);
    ...
end
```



## 引用传递

```verilog
bit[2:0] port_a,port_b;
covergroup CovPort(ref bit[2:0]port, input int mid);
    coverpoint port{
        bins lo={[0:mid-1]};
        bins hi={mid:$};
    }
endgroup
CovPort cpa,cpb;
initial begin
    cpa=new(port_a,4);
    cpb=new(port_b,2);
    ...
end
```



# 7. 覆盖选项

SV提供两类：

- 实例选项
- 类型选项



- 单个实例的覆盖率

  同一个覆盖组可以多次实例化；

  若需查看单个实例的覆盖率报告，需要进行独立实例化

  关键词：`per_instance`

  ```verilog
  covergroup CovLen;
      coverpoint tr.len;
      option.per_instance=1;
      // 在注释中使用层次化路径
      option.comment=$psprintf("%m");
  endgroup
  ```

  

- 覆盖组注释

  ```verilog
  covergroup CovPort;
      type_option.comment="Port numbers";
      coverpoint port;
  endgroup
  ```

  