- [1. OOP术语](#1-oop术语)
- [2. 创建对象](#2-创建对象)
  - [构造函数](#构造函数)
  - [为对象创建句柄](#为对象创建句柄)
  - [对象的回收](#对象的回收)
  - [对象的使用](#对象的使用)
- [3. 静态变量和动态变量](#3-静态变量和动态变量)
  - [静态变量](#静态变量)
  - [静态方法](#静态方法)
- [4. 类的方法](#4-类的方法)
  - [类之外定义方法](#类之外定义方法)
- [5. 访问权限（public/protected/local）](#5-访问权限publicprotectedlocal)
- [6. 任务和函数](#6-任务和函数)
  - [**函数和任务区别**](#函数和任务区别)
  - [参数传递：ref使用](#参数传递ref使用)
- [7. 类的三要素](#7-类的三要素)
  - [封装](#封装)
  - [继承](#继承)
    - [扩展类的构造函数](#扩展类的构造函数)
    - [扩展类中的约束](#扩展类中的约束)
  - [多态](#多态)
    - [虚方法](#虚方法)
    - [纯虚方法](#纯虚方法)
- [8. this和super](#8-this和super)
  - [this](#this)
  - [super](#super)
  - [子类和父类具有相同的变量名](#子类和父类具有相同的变量名)
- [9. $cast](#9-cast)
- [10. 类中包含另一个类](#10-类中包含另一个类)
  - [编译顺序](#编译顺序)
- [11. 对象的复制](#11-对象的复制)
  - [使用new操作符：浅拷贝](#使用new操作符浅拷贝)
  - [深拷贝：自己编写复制函数](#深拷贝自己编写复制函数)
- [12. 参数化的类](#12-参数化的类)

# 1. OOP术语

- OOP

  **Object-Oriented Programming**（面向对象编程）



- 类：变量和子程序的基本构建块。与之对应的是module

- 对象object：类的一个实例

- 句柄（handle）：指向对象的指针

- 属性（property）：存储数据的变量

- 方法（method）：任务或函数中操作变量的代码

- 原型（prototype）：程序的头，包括程序名、返回类型和参数列表



- 类的编写

  类：封装数据和操作的子程序

```verilog
class Trans;
    bit [31:0] addr,crc,data[8];
    function void display;
        $display("Trans:%h", addr);
    endfunction:display
    
    function void cal_crc;
        crc=addr^data.xor;
    endfunction:cal_crc
endclass:Trans
```



- 类与模块

  - module是静态的；class是动态的，可随时创建或销毁对象；
  - module的实例名只能指向一个实例；class的句柄可以指向多个对象，但是一次只能指向一个，这个和class是动态的密切相关

  - module中可以定义class，但class中不能包含module;



# 2. 创建对象

- 声明和使用句柄

  ```verilog
  Trans tr; //声明一个句柄，初始化为null值
  Tr=new(); //为Trans对象分配空间，变量初始化为默认值，并返回对象的地址
  ```

  

## 构造函数

**new()函数**：分配内存，初始化对象中的变量

- 自定义new()函数：将addr和data初始化为固定值

```verilog
class Trans;
    logic [31:0] addr,crc,data[8];
    
    function new;
        addr=3;
        foreach(data[i]) data[i]=5;
    endfunction
endclass
```

- 带参数的new()函数

```verilog
class Trans;
    logic [31:0] addr,crc,data[8];
    
    function new(logic [31:0] a=3,d=5);
        addr=a;
        foreach(data[i]) data[i]=d;
    endfunction
endclass
initial begin
    Trans tr;
    tr=new(10); // addr使用传递的值10 data使用默认值
end
```



- 声明句柄和创建对象时应该分开操作



- new()和new[]的区别

  二者均用于申请内存和初始化变量

  new()函数：仅创建一个对象，可以使用多个参数

  new[]操作：建立一个含多个元素的数组，只可使用一个数值，用于设置动态数组大小



## 为对象创建句柄

- 句柄和对象

  先声明句柄，再创建对象；

  一个句柄可以指向多个对象

```verilog
Trans t1,t2;
t1=new(); //为第一个对象分配地址
t2=t1; //t2和t1都指向第一个对象
t1=new(); //t1指向第二个创建的对象
```



## 对象的回收

使用null回收内存

```verilog
Trans t;
t=new();
t=new(); //分配第二个对象，并释放第一个对象的内存
t=null； //释放第二个对象的内存
```

- 句柄和C语言区别

  - SV不允许对句柄作和C语言类似的改变，也不允许将一种类型的句柄指向另一种类型的对象。

  - SV中在没有任何句柄指向一个对象的时候，自动回收对象占用的内存，保证了句柄的合法性；

    C/C++的指针可以指向一个不存在的对象，当忘记手动释放对象时，代码可能存在内存泄漏



## 对象的使用

- 使用 “.” 符号来引用变量和子程序

```verilog
Trans t;
t=new();
t.addr=32'h42;
t.display();
```



# 3. 静态变量和动态变量

## 静态变量

- 特点：被所有实例对象共享，作用范围仅限于该类;

- 不论创建多少个对象，静态变量count只有1个;
- 变量id不是静态的，所以每个Trans都有自己的id变量；

```verilog
class Trans;
    static int count=0; //指示创建对象的数目
    int id;
    function new();
        id=count++;
    endfunction
endclass

Trans t1,t2;
initial begin
    t1=new(); //第一个实例 id=0,count=1
    t2=new(); //第二个实例 id=1,count=2·	
    $display("Second id=%d,count=%d",t2.id,t2.count);
end
```



- 无需句柄，通过类名+`::`，即类作用域操作符，也可以访问静态变量

- 静态变量：属于类本身，不需要实例化也可以直接使用

- 初始化

  静态变量在声明时初始化，不能在构造函数中初始化



## 静态方法

- 在类中创建静态方法读写静态变量

```verilog
class Trans;
    static Config cfg;  //静态句柄声明
    static int count=0;
    int id;
    //显示静态变量的静态方法
    static function void displat_statics();
        $display("Trans cfg.mode=%s,count=%0d", cfg.mode.name(),count);
    endfunction
endclass

Config cfg;
initial begin
    cfg=new(MODE_ON);
    Trans::cfg=cfg;
    Trans::display_statics(); //无需句柄和创建对象，即可调用静态方法
end
```



- SV中不允许非静态方法读写非静态变量，例如id



# 4. 类的方法

- 类的方法：即类的作用域内定义的task或function

- **==<u>默认使用自动存储</u>==**，可以不用automatic修饰



## 类之外定义方法

关键词：`extern`

```
class Trans;
    bit [31:0] addr,crc,data[8];
    extern function void display();
endclass

function void Trans::display();
	$display("@%0t:Trans addr=%h,crc=%h",$time,addr,crc);
	$write("\tdata[0-7]=");
	foreach(data[i]) $write(data[i]);
	$dislpay();
endfunction
```



# 5. 访问权限（public/protected/local）

- public：公开访问，无任何限制
- protected：受保护，允许类内部及其子类访问
- local：私有，只允许类内部访问，子类不可见

> 默认访问权限为public



# 6. 任务和函数

- 函数function

  默认有返回值和返回类型，默认返回类型为logic，未指定返回值位宽时，默认为1bit

  不需要返回值时，定义：`function void`

```verilog
function [7:0]add;
    input [7:0] a,b;
    add = a+b;
endfunction

function [7:0]add;
    input [7:0] a,b;
    return a+b;
endfunction
```





- 任务task

  无返回值

  - 参数缺省输入方向时，默认为输入`input`方向

  - 参数缺省类型时，默认为`logic`类型



## **函数和任务区别**

- 任务中可以有时间控制语句（`#, @, wait`）；函数不能

- 任务可以调用函数，函数不能调用任务
- 函数可以在表达式中调用（如赋值右侧），任务必须独立使用
- 执行时间：函数立即执行，零仿真时间；任务为耗时操作
- 返回值：函数只能返回1个值，任务可以通过output输出多个值



## 参数传递：ref使用

Verilog对参数的处理方式：复制

- 在子程序开头，把input和inout的值复制给本地变量
- 在子程序退出时，复制output和inout的值

另一种传递方式：引用

- ref参数

  - 直接传递变量的引用，函数/任务内对参数的修改会直接影响外部变量的值
  - 避免大型数据的复制开销
  - 参数方向隐含为`inout`类型

- 常量引用：`const ref`

  传递只读引用，子程序内部无法修改参数值，但避免了复制开销



# 7. 类的三要素

## 封装

数据和操作作为一个整体单元



## 继承

基于已存在的类，创建一个新的类（派生类/子类）

子类继承父类所有public和protected成员；

可添加自己的成员，或改写父类的方法



- 原始类：父类/超类
- 扩展类：派生类/子类
- 基类：不从任何其他类派生得到的类



### 扩展类的构造函数

- 基类的构造函数有参数，扩展类必须有一个构造函数，且必须在构造函数的第一行调用基类的构造函数

```verilog
class Base;
    int var;
    function new(input int var);
        this.var=var;
    endfunction
endclass

class Extended extends Base;
    function new(input int var);
        super.new(var);
        //构造函数的其他行为
    endfunction
endclass
```



### 扩展类中的约束

- 约束块同名：子类约束取代父类约束，父类约束块不再生效；

```verilog
class Base;
    rand bit [7:0] addr;
    constraint c_range { addr < 10; } // 父类约束：小于 10
endclass

class Child extends Base;
    constraint c_range { addr > 100; } // 名字相同！完全替代父类
endclass

// 结果：addr 最终会在 101~255 之间随机，完全无视 < 10 的规则。
```



- 约束块异名：子类和父类约束同时生效，求交集

```verilog
class Base;
    rand bit [7:0] addr;
    constraint c_range_base { addr < 50; }
endclass

class Child extends Base;
    constraint c_range_child { addr > 20; }
endclass

// 结果：两个约束同时起作用，addr 最终在 21~49 之间随机。
```





## 多态

同一操作可以根据作用对象的不同类型而表现不同的行为；

- 虚方法：`virtual`

- 方法重写

  子类使用相同的签名（方法名、参数、返回类型）重新定义父类的virtual方法

- **父类句柄指向子类对象**

```verilog
module po;
    class Packet;
        int crc;
        extern virtual function int compute_crc();
   	endclass
   	
   	function int Packet::compute_crc();
   		return 1;
   	endfunction
   	
   	class MyPacket extends Packet;
        extern virtual function int compute_crc();
   	endclass
   	
   	function int MyPacket::compute_crc();
   		return 0;
   	endfunction

	function int crc(Packet pkt);
		crc = pkt.compute_crc();
	endfunction

	initial begin
		Packet p1;
		MyPacket p2;
		p1=new();
		p2=new();
		p1.crc = crc(p1);
        $display("p1.crc = %0d", p1.crc); // p1.crc = 1
        p2.crc = crc(p2); // 父类句柄Packet指向了子类对象p2
        $display("p2.crc = %0d", p2.crc); // p2.crc = 0
	end
endmodule
```

上述代码:

- 父类使用了virtual方式定义了方法compute_crc()；
- 子类MyPacket继承了父类的crc变量，并改写了父类的方法
- 当父类句柄指向了子类对象时，子类重写的方法会生效



### 虚方法

- 使用virtual

  SV 根据对象类型，而非句柄类型决定调用的方法是来自子类还是父类；

- 没有使用virtual

  SV根据句柄的类型，决定调用的方法来自子类还是父类



虚方法要求：

- 子类和父类的签名相同

  类型，参数



### 纯虚方法

- 特点：只有声明，无具体实现

  - 必须在虚类中定义
  - 无法被实例化，只能被继承

  - 可以声明句柄，但是不能创建该类的对象；

  - 子类必须重写纯虚方法，除非子类也是虚类；

- 关键词：`pure virtual`

```verilog
// 1. 必须定义在虚拟类（抽象类）中
virtual class BaseDriver;
    
    // 2. 纯虚方法：只有名字、参数和返回值，没有逻辑
    pure virtual function void drive_bus(); 
    
endclass

class MyDriver extends BaseDriver;

    // 3. 子类必须实现这个方法，否则 MyDriver 无法被实例化
    virtual function void drive_bus();
        $display("Driving the bus with specific logic...");
    endfunction

endclass
```





# 8. this和super

## this

```verilog
class Scoping;
    string oname;
    function new(string oname);
        this.oname = oname; // 类变量 = 局部变量（传递进来的局部变量）
    endfunction
endclass
```



## super

- 用途：引用父类的成员
- 使用场景
  - 子类重写父类方法时，调用父类的原始方法
  - 访问父类中被隐藏的成员变量
  - 在子类构造函数中调用父类的构造函数



## 子类和父类具有相同的变量名

> 和虚方法需要进行区分

- 子类内部访问：直接访问子类的，用`super.变量名`访问父类的
- 外部访问：取决于句柄的类型

```verilog
class Base;
    int data = 10;
endclass

class Child extends Base;
    int data = 20; // 隐藏了父类的 data
    
    function void print();
        $display("Child data: %0d", data); // 输出 20
    endfunction
    
    function void print_both();
        $display("Child data: %0d", data);       // 访问子类的变量 (20)
        $display("Base data: %0d", super.data); // 访问父类的变量 (10)
    endfunction
endclass

Child c = new();
Base  b = c; // 父类句柄指向子类对象

initial begin
    $display("c.data = %0d", c.data); // 输出 20 (子类句柄)
    $display("b.data = %0d", b.data); // 输出 10 (父类句柄)
end
```



# 9. $cast

- **类型向下转换或类型变换**

  - 将一个指向基类的指针转换为一个指向派生类的指针；

    这里说的是：父类句柄可以指向子类对象，或者说子类句柄可以赋值给父类句柄；

  ```verilog
  Trans tr; //声明父类/基类句柄
  BadTr bad; //子类句柄
  bad=new(); //创建子类对象
  tr=bad;
  ```

  

  - 基类/父类对象赋值给子类句柄：**操作会失败**，但不一定总是非法的

    **合法**情况：基类句柄确实指向一个子类/派生类对象

  - `$cast`子程序：检查句柄所指向的对象类型，合法返回非零值，非法返回0

  ```verilog
  bad=new();
  tr=bad;
  
  if($cast(bad2, tr)) $display(bad2.bad_crc);
  ```

  

# 10. 类中包含另一个类

使用场景：太大的类分成若干小类

- Stat类

```verilog
class Stat;
    time startT,stopT; //事务时间
    static int ntrans=0; //事务数目
    static time total_elapsed_time=0;
    
    function time how_long;
        how_long=stopT-startT;
        ntrans++;
        total_elapsed_time+=how_long;
    endfunction
    function void start;
        startT=$time;
    endfunction
endclass
```

- Trans类

```verilog
class Trans;
    bit [31:0] addr,crc,data[8];
    Stat stats; //Stat句柄声明
    
    function new();
        stats=new(); //创建对象
    endfunction
    
    task create_packet();
        stats.start();
    endtask
endclass
```



## 编译顺序

编译类时，容易遇到尚未定义的类，声明这种被包含的类的句柄会引起错误

- 解决方法：typedef

```verilog
typedef class Stat;
class Trans;
    Stat stats;
    ...
endclass
class Stat;
    ...
endclass
```



# 11. 对象的复制

## 使用new操作符：浅拷贝

```verilog
class Trans;
    bit [31:0] addr,crc,data[8];
endclass

Trans src,dst;
initial begin
    src=new(); //创建对象
    dst=new src; //对象复制
end
```

- 缺陷

  如果类中包含**其他类的句柄**，浅拷贝只会复制“句柄”（指针），而不会复制“指向的对象”。这意味着两个父对象会共享同一个子对象。

  如：Trans若包含Stat类，复制后，两个Trans类型的对象共享同一个Stat对象

  ```verilog
  class Trans;
      bit [31:0] addr,crc,data[8];
      static int count=0;
      int id;
      Stat stats;
      
      function new();
          stats=new();
          id=count++;
      endfunction  
  endclass
  Trans src,dst;
  initial begin
      src=new();
      src.stats.startT=42;
      dst=new src;
      dst.stats.startT=96; //改变了dst和src的stats，二者共享一个stats对象
      $display(src.stats.startT); //96
  end
  ```



## 深拷贝：自己编写复制函数

SV中没有内置的深拷贝方法，需要自己编写

```verilog
class Trans;
    bit [31:0] addr,crc,data[8];
    static int count=0;
    int id;
    Stat stats;
    
    function new();
        stats=new();
        id=count++;
    endfunction  
    
    function Trans copy;
        copy=new();
        copy.addr=addr;
        copy.crc=crc;
        copy.data=data;
        copy.stats=stats.copy(); //调用Stat::copy函数
        id=count++;
    endfunction
endclass

class Stat;
    time startT,stopT;
    ...
    function Stat copy();
        copy=new();
        copy.startT=startT;
        copy.stopT=stopT;
    endfunction
endclass
```



# 12. 参数化的类

和参数化的module类似

- 数值参数

```verilog
class Vector #(parameter int WIDTH = 8); // 默认宽度为 8
    bit [WIDTH-1:0] data;
endclass

// 实例化时：
Vector #(16) v16 = new(); // 宽度变为 16
Vector #(.WIDTH(32)) v32 = new(); // 显式指定宽度为 32
```



- 类型参数

```verilog
class Container #(type T = int); // 默认存储 int 类型
    T item;
endclass

// 实例化：
Container #(string) c1 = new(); // 此时 item 是 string 类型
Container #(bit [7:0]) c2 = new(); // 此时 item 是 8 位 bit
```



注意：

- 参数不同的类，被视为完全不同的类型

```verilog
Vector #(8)  va;
Vector #(16) vb;

// va = vb; // 编译错误！虽然都来自同一个类模板，但它们类型不兼容。
```



- 对于静态成员，**每一组不同的参数都会产生一套独立的静态变量**

```verilog
class Box #(int ID = 0);
    static int count = 0;
endclass

initial begin
    Box #(1) b1_a = new();
    Box #(1) b1_b = new();
    Box #(2) b2 = new();
    
    b1_a.count++; // Box#(1) 的 count 变成 1
    // 此时 b1_b.count 也是 1（因为它们参数相同）
    // 但 b2.count 依然是 0（因为它是 Box#(2)，属于不同的特化类型）
end
```

