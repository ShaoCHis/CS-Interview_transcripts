# 面试真题
- [面试真题](#----)
  * [1、腾讯-天美工作室群IEG-一面 2025.3.6](#1-腾讯-天美工作室群IEG一面2025.3.6)
    + [1.1、protobuf序列化和反序列化过程，json和protobuf的使用场景](#11-protobuf序列化和反序列化过程，json和protobuf的使用场景)
    + [1.2、如果查看一个已经运行的进程中某一个线程CPU的占用率？代码怎么大致计算？](#12-如果查看一个已经运行的进程中某一个线程CPU的占用率？代码怎么大致计算？)
    + [1.3、如何查看一个进程已经打开的文件描述符的数量](#13-如何查看一个进程已经打开的文件描述符的数量)
    + [1.4、手撕循环队列](#14-手撕循环队列)


## 1、腾讯-天美工作室群IEG-一面 2025.3.6

### 1.1、protobuf序列化和反序列化过程，json和protobuf的使用场景

Protobuf序列化原理简介：

> 序列化：序列化是将数据结构或对象转换成二进制字节流的过程。
> Protobuf对于不同的字段类型采用不同的编码方式和数据存储方式对消息字段进行序列化，以确保得到高效紧凑的数据压缩。
> Protobuf序列化过程如下：
> (1)判断每个字段是否有设置值，有值才进行编码。
> (2)根据字段标识号与数据类型将字段值通过不同的编码方式进行编码。
> (3)将编码后的数据块按照字段类型采用不同的数据存储方式封装成二进制数据流。
> ————————————————原文链接：https://blog.csdn.net/weixin_43971373/article/details/119729776
>
> 1.2反序列化
> 反序列化是将在序列化过程中所生成的二进制字节流转换成数据结构或者对象的过程。
> Protobuf反序列化过程如下：
> (1)调用消息类的parseFrom(input)解析从输入流读入的二进制字节数据流。
> (2)将解析出来的数据按照指定的格式读取到C++、Java、Phyton对应的结构类型中。

Protobuf数据存储方式

>T-L-V(Tag - Length - Value)，即标识符-长度-字段值的存储方式，其原理是以标识符-长度-字段值表示单个数据，最终将所有数据拼接成一个字节流，从而实现数据存储的功能。
>其中Length可选存储，如储存Varint编码数据就不需要存储Length，此时为T-V存储方式。
>
>T-L-V 存储方式的优点：
>A、不需要分隔符就能分隔开字段，减少了分隔符的使用。
>B、各字段存储得非常紧凑，存储空间利用率非常高。
>C、如果某个字段没有被设置字段值，那么该字段在序列化时的数据中是完全不存在的，即不需要编码，相应字段在解码时才会被设置为默认值。
>
>**Tag是消息字段标识符和数据类型经Varint与Zigzag编码后的值**，因此Tag存储了字段的标识符(field_number)和数据类型(wire_type)，即Tag = 字段数据类型(wire_type) + 标识号(field_number)。

![img](https://i-blog.csdnimg.cn/blog_migrate/f43ec9e9222d3b1cfd5925c3aec8d524.png)

1. 先取到message的discriptor
2. 根据message的discriptor，取出所有field的discriptor------》放入vector中
3. 有了message和field，去找定给定field的所有字段，实际上得到的是字段到message的偏移量
4. 那么根绝message的首地址和偏移量，再将其转换为字段的类型，就得到了这个字段的指针
5. 再解引用就得到值

**前向兼容和后向兼容：**：

> 对于服务端返回字段不确定的情形，有几种方法可以处理，而无需频繁升级客户端
>
> 1. 使用 **Any** 类型，可以将不确定的字段封装在 Any 类型中，这样即使客户端添加了新的字段类型，客户端也可以解析已知字段，而忽略不认识的 Any字段
> 2. 使用 **oneof** 结构，如果返回的类型是有限的几种，可以使用 oneof 来定义这些可能的字段，这样客户端可以解析它认识的字段，并忽略不认识的字段
> 3. 扩展字段：Protobuf也支持扩展字段，这允许你在不破坏向后兼容性的情况下添加新的字段，客户端可以忽略他不理解的扩展字段
> 4. 使用包装类型，对于可选的字段，可以使用Protobuf的包装类型，如 google.protobuf.StringValue、Int32Value等，这样即使某些字段在某些情况下不被设置，也不会破坏消息的格式

toString---》二进制字段写值，**属性的名称**实际上对应的是**message中字段的id**，动态反射（基类调用其他类的方法）。

**应用场景：**

1. 客户端<-->服务端，为了方便灵活，比如 服务端返回的字段，经常更新，不可能要求客户端一并更新，这时协议是松散的，适合用json
2. 服务端1<-->服务端2，服务端是由开发人员维护的，可以做到同时一并升级，服务端的RPC框架传输，可以使用protobuf

<span style="color:red">Protobuf不支持动态解析，在编码和解码时需要预先定义数据结构，因此它不支持像JSON那样可以动态地解析任意结构的数据。这在处理一些需要灵活处理数据结构的场景时可能会受到限制。这项绝对导致，他不适合客户端<-->服务端之间的通讯，因为接口会经常更新，变化，不能动态适应，导致客户端也需要升级，是不现实的。</span>

JSON灵活，Protobuf高效，性能好，体积小。



### 1.2、如果查看一个已经运行的进程中某一个线程CPU的占用率？代码怎么大致计算？

使用Linux系统命令查看

```shell
# 1.首先查看 PID
#在最后一栏中可以看到进程名和进程的PID
netstat -tnlp -all
#或者第四列就是进程的PID
ps -ef -all

# 2.通过PID查看该进程下，运行的线程号SPID
ps -T -p 进程ID 
#或者
ps -eLf|grep 进程PID



#或者直接通过top命令查看，第一列是进程号是线程的PID
top -H -p 进程PID


#通过树状结构查看
pstree -p 5010
```



代码实现：

> 通过读取/proc/stat文件获取当前系统的CPU占用率。
>
> > proc是一个只存在内存当中的伪文件系统，他以文件系统的方式为内核与进程提供通信的接口。用户和应用程序可以通过/proc得到动态的从系统内核读出的系统的信息，并可以改变内核的某些参数
>
> > /proc/stat 包含了系统启动以来的许多关于kernel和系统的统计信息, 其中包括**CPU运行情况**、中断统计、启动时间、上下文切换次数、运行中的进程等等信息。其实，/proc/stat反映的就是CPU总的占用时间，如下图所示。
> >
> > ![Alt](https://i-blog.csdnimg.cn/blog_migrate/6e957daa670bd41eeb65872fa366ca4c.png#pic_center)
> >
> > **参数      解析（单位：jiffies）**
> >
> > (jiffies是内核中的一个全局变量，用来记录自系统启动一来产生的节拍数，在linux中，一个节拍大致可理解为操作系统进程调度的最小时间片，不同linux内核可能值有不同，通常在1ms到10ms之间)
> >
> > **user** (12939774)  从系统启动开始累计到当前时刻，处于用户态的运行时间，不包含 nice值为负进程。
> >
> > **nice** (3709)   从系统启动开始累计到当前时刻，nice值为负的进程所占用的CPU时间
> >
> > **system** (18395244) 从系统启动开始累计到当前时刻，处于核心态的运行时间
> >
> > **idle** (842761120)  从系统启动开始累计到当前时刻，除IO等待时间以外的其它等待时间**iowait** (17500556) 从系统启动开始累计到当前时刻，IO等待时间(since 2.5.41)
> >
> > **irq** (0)      从系统启动开始累计到当前时刻，硬中断时间(since 2.6.0-test4)
> >
> > **softirq** (49424)   从系统启动开始累计到当前时刻，软中断时间(since2.6.0-test4)
> >
> > **stealstolen**(0)           which is the time spent in otheroperating systems when running in a virtualized environment(since 2.6.11)
> >
> > **guest**(0)                whichis the time spent running a virtual CPU for guest operating systems under the control ofthe Linux kernel(since 2.6.24)

**CPU利用率的计算**

CPU利用率分为用户态，系统态和空闲态，分别表示CPU处于用户态执行的时间，系统内核执行的时间，和<span style="color:red">空闲系统进程</span>执行的时间。平时所说的CPU利用率是指：CPU执行非系统空闲进程的时间 / CPU总的执行时间。

通过线程ID获得各线程的CPU使用率。

主要是通过分析**/proc/<pid>/task/<tid>/stat**文件获得，pid为程序的PID，tid为程序的各个线程的ID号（就是第一步输出的线程ID），stat文件就是一些调度的基本信息，具体可参阅：Linux proc/pid/task/tid/stat文件详解 http://blog.csdn.net/ctthuangcheng/article/details/18090701

线程比较多的时候一个线程一个线程去分析该文件比较费劲，可通过脚本一次解析完成，参数为进程PID，运行成功会输出该进程的所有线程tid、用户层CPU使用、内核态CPU使用，数值越高表示消耗CPU资源越多。

```shell
# 嵌入线程中进行线程CPU各占用率的输出
#!/bin/sh
#get /proc/pid/task/tid/stat
#$1 is tid
#$14  is user cpu 
#$15 is sys cpu
echo "tid user sys"
for file in /proc/$1/task/*
do
    if test -d $file
    then
        cat $file/stat | awk -F" " '{print $1 " " $14 " " $15}'
    fi
done
```

**线程Cpu时间threadCpuTime = utime +stime**



### 1.3、如何查看一个进程已经打开的文件描述符的数量

**获取系统打开的文件描述符数量**

```sh
  [root@localhost ~]# cat /proc/sys/fs/file-nr 

  1216  0    197787

   //第一列 1216  ：为已分配的FD数量

   //第二列 0     ：为已分配但尚未使用的FD数量

   //第三列197787：为系统可用的最大FD数量

  已用FD数量＝为已分配的FD数量 - 为已分配但尚未使用的FD数量。注意，这些数值是系统层面的。
```

**获取进程打开的文件描述符数量**

```sh
    [root@localhost ~]# pidof vim

      3253

    [root@localhost ~]# ll /proc/3253/fd

     总用量 0

    lrwx------. 1 test test 64  6月  8 18:11 0 -> /dev/pts/0

    lrwx------. 1 test test 64  6月  8 18:11 1 -> /dev/pts/0

    lrwx------. 1 test test 64  6月  8 18:11 2 -> /dev/pts/0

    lrwx------. 1 test test 64  6月  8 18:11 4 -> /home/test/.bash_history.swp

    //可以看到vim进程用了4个FD
```

**获取整个系统打开的文件数量**

```sh
 [root@localhost ~]# lsof |wc -l

    1864
```

**获取某个用户打开的文件数量** 

```sh
  [root@localhost ~]# lsof -u test |wc -l

   15
```

**获取某个程序打开的文件数量** 

```sh
  [root@localhost ~]# pidof vim

   3253

  [root@localhost ~]# lsof -p 3253 |wc -l

   31
```

上面所示只是用lsof来显示已打开的文件数量，lsof的功能远不止这些，有兴趣可以man lsof看一下 

### 1.4、手撕循环队列

```c++
#define MAX_SIZE 10

class Queue{
public:
  Queue(){
    arr_ = new int[MAX_SIZE];
    head_=0;
    tail_=0;
  }
  
  ~Queue(){
    delete[] arr_;
  }
  
  void insert(int&& val){
    arr_[tail_] = val;
    tail_ = (tail_+1)%MAX_SIZE;
  }
  
  int get(){
    int temp = arr_[head_];
    head_ = (head_+1)%MAX_SIZE;
    return temp;
  }
  
  bool isFull(){
    return (tail_+1)%MAX_SIZE == head_;
  }
  
  bool isEmpty(){
    return head_==tail_;
  }
  
private:
  int *arr_;
  int head_,tail_;
}
```

