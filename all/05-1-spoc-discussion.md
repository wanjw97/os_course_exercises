#lec11: 进程／线程概念spoc练习

## 视频相关思考题

### 11.1 进程的概念

1. 什么是程序？什么是进程？
程序是静态代码
进程是一个程序在一个数据集合上的一次动态执行过程

2. 进程有哪些组成部分？
程序、数据、执行状态

3. 请举例说明进程的独立性和制约性的含义。
独立性是指进程是资源分配和调度的基本单位
制约性是指进程间有共享资源和通信

4. 程序和进程联系和区别是什么？
程序是进程的组成部分，同一个程序可以有多个进程
进程是动态的，程序是静态的。进程是暂时的，程序是永恒的

### 11.2 进程控制块

1. 进程控制块的功能是什么？
管理和控制进程运行

2. 进程控制块中包括什么信息？
进程基本信息
进程控制信息
运行状态

3. ucore的进展控制块数据结构定义中哪些字段？有什么作用？


### 11.3 进程状态

1. 进程生命周期中的相关事件有些什么？它们对应的进程状态变化是什么？
创建、执行、等待、抢占、唤醒、结束
创建、就绪、执行、等待、退出

### 11.4 三状态进程模型

1. 运行、就绪和等待三种状态的含义？7个状态转换事件的触发条件是什么？
运行：占用CPU执行指令

就绪：已获取所有其他资源，等待CPU资源

等待：等待某事件，处理暂停状态

转换事件：启动、进入就绪队列、被调度、等待事件、时间片用完、事件发生、结束

### 11.5 挂起进程模型

1. 引入挂起状态的目的是什么？
减少内存占用

2. 引入挂起状态后，状态转换事件和触发条件有什么变化？
多了两个变迁：激活和挂起

3. 内存中的什么内容放到外存中，就算是挂起状态？
进程内核栈

### 11.6 线程的概念

1. 引入线程的目的是什么？
同一地址空间内实现并发

2. 什么是线程？
线程是进程中描述指令流执行状态的组成部分，是CPU调度的基本单位

3. 进程与线程的联系和区别是什么？

### 11.7 用户线程

1. 什么是用户线程？
用户函数库提供线程实现

2. 用户线程的线程控制块保存在用户地址空间还是在内核地址空间？
用户地址空间

### 11.8 内核线程

1. 用户线程与内核线程的区别是什么？
内核线程是系统调用实现的线程机制

2. 同一进程内的不同线程可以共用一个相同的内核栈吗？
用户线程可以

3. 同一进程内的不同线程可以共用一个相同的用户栈吗？
不可以

## 选做题
1. 请尝试描述用户线程堆栈的可能维护方法。

## 小组思考题
(1) 熟悉和理解下面的简化进程管理系统中的进程状态变化情况。
 - [简化的三状态进程管理子系统使用帮助](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep7-process-run.md)
 - [简化的三状态进程管理子系统实现脚本](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep7-process-run.py)

(2) (spoc)设计一个简化的进程管理子系统，可以管理并调度如下简化进程。在理解[参考代码](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab4/process-concept-homework.py)的基础上，完成＂YOUR CODE"部分的内容。然后通过测试用例和比较自己的实现与往届同学的结果，评价自己的实现是否正确。可２个人一组。

### 进程的状态

 - RUNNING - 进程正在使用CPU
 - READY   - 进程可使用CPU
 - DONE    - 进程结束

### 进程的行为
 - 使用CPU,
 - 发出YIELD请求,放弃使用CPU


### 进程调度
 - 使用FIFO/FCFS：先来先服务,
   - 先查找位于proc_info队列的curr_proc元素(当前进程)之后的进程(curr_proc+1..end)是否处于READY态，
   - 再查找位于proc_info队列的curr_proc元素(当前进程)之前的进程(begin..curr_proc-1)是否处于READY态
   - 如都没有，继续执行curr_proc直到结束

### 关键模拟变量
 - 进程控制块
```
PROC_CODE = 'code_'
PROC_PC = 'pc_'
PROC_ID = 'pid_'
PROC_STATE = 'proc_state_'
```
 - 当前进程 curr_proc
 - 进程列表：proc_info是就绪进程的队列（list），
 - 在命令行（如下所示）需要说明每进程的行为特征：（１）使用CPU ;(2)等待I/O
```
   -l PROCESS_LIST, --processlist= X1:Y1,X2:Y2,...
   X 是进程的执行指令数;
   Ｙ是执行CPU的比例(0..100) ，如果是100，表示不会发出yield操作
```
 - 进程切换行为：系统决定何时(when)切换进程:进程结束或进程发出yield请求

### 进程执行
```
instruction_to_execute = self.proc_info[self.curr_proc][PROC_CODE].pop(0)
```

### 关键函数
 - 系统执行过程：run
 - 执行状态切换函数:　move_to_ready/running/done　
 - 调度函数：next_proc

### 执行实例

#### 例１
```
$./process-simulation.py -l 5:50
Process 0
  yld
  yld
  cpu
  cpu
  yld

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN YIELD
Time     PID: 0
  1     RUN:yld
  2     RUN:yld
  3     RUN:cpu
  4     RUN:cpu
  5     RUN:yld

```


#### 例２
```
$./process-simulation.py  -l 5:50,5:50
Produce a trace of what would happen when you run these processes:
Process 0
  yld
  yld
  cpu
  cpu
  yld

Process 1
  cpu
  yld
  cpu
  cpu
  yld

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN YIELD
Time     PID: 0     PID: 1
  1     RUN:yld      READY
  2       READY    RUN:cpu
  3       READY    RUN:yld
  4     RUN:yld      READY
  5       READY    RUN:cpu
  6       READY    RUN:cpu
  7       READY    RUN:yld
  8     RUN:cpu      READY
  9     RUN:cpu      READY
 10     RUN:yld      READY
 11     RUNNING       DONE
```
