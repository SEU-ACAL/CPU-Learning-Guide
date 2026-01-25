# CPU设计入门（劝退）指南

# 前言

1.  整个CPU设计就是一个trade off
    
2.  CPU设计付出和收益不成正比，性价比很低
    
3.  对于CPU设计这个领域，**语言（sv，chisel）**并不重要，对**微架构**的理解才是最重要的
    
4.  想了解整个CPU核心的设计是一个长期的工程，需要付出大量的时间成本，所以尽量先聚焦某个模块（BPU,Cache...）
    
5.  写模拟器时要想可不可以落地，尽量不要太偏理想化
    
6.  要去搜寻业界CPU设计的文档/发布会PPT或者逆向博客，了解业界的主流方向
    

> 实际上，只要是处理器设计上面几点都是成立的

> 对于2，语言并不重要，但数字电路一定要熟悉

# 基础部分

对于**大三大四**的同学而言，如果先前没有CPU设计基础，强烈建议学习ysyx：[一生一芯](https://ysyx.oscc.cc/)

> 判定标准：是否写过五级流水线，并且知道一个hello world如何在裸机CPU运行的

> 实际上，不管是学习GPU，NPU设计，都建议去先学习ysyx打好基础

需要完成整个**B阶段**，总共耗时大概**半年**（和个人基础相关）

对于研一的同学，如果基础比较薄弱，依然先推荐ysyx

需要完成整个**C阶段，**总共耗时**3个月**

## 数字电路设计注意点

1.  不要以软件思维写硬件代码，不一定要对整体模块进行门级建模，但一定要知道自己写出的RTL大概是一个什么数字电路结构
    
2.  尽量不要去对SRAM读口的数据进行逻辑操作，尽量打拍后操作
    
3.  一级流水线尽量少于40级逻辑
    
4.  如果发现时序报告有很多buf/inv造成的延迟很大，需要对某些信号进行复制操作，以减少扇出（时序报告也有扇出数据）
    
5.  时序违例最简单的解决办法是打拍
    

# 进阶部分

首先推荐自己的知识库（整理了近20年四大顶会的文章，预取可能少一些）：

[https://ima.qq.com/wiki/shareId=a5e052ac7d7c54128f3ab682bd2482d825973edf98186645533867142ecaf092](https://ima.qq.com/wiki/?shareId=a5e052ac7d7c54128f3ab682bd2482d825973edf98186645533867142ecaf092)

**也欢迎大家完善知识库**

> **之后的方向均可在该知识库找到文章**

## 基础知识

**需要先通读一遍：超标量处理器设计（姚勇斌）**

高性能CPU设计目前可以分为四个部分：前端、后端、访存、缓存

其中前端主要是供应指令，后端是对指令进行调度和执行，访存主要是和L1DCache交互，缓存就是L2,L3Cache

下面会列出CPU核内所有的方向（maybe）：

### 前端

1.  分支预测
    
    1.  TAGE（业界标配，目前只有tage，近年文章全是在TAGE上改进）
        
    2.  H2P分支预测器
        
    3.  分支预测延迟对性能影响
        
    4.  分支预测惩罚对性能影响（业界一般10-12，intel和AMD可能多一些，其对性能影响较大）
        
    
    > 现在分支预测一般使用override 预测器结构
    
2.  指令预取
    
    1.  基于FDIP（业界大部分做法）
        
    2.  基于time stream
        
    3.  错误路径预取
        
3.  BTB
    
    1.  BTB三种常见结构（IBTB,BBTB,RBTB，业界一般RBTB）
        
    2.  BTB预取（预取到原BTB（修改过的结构） or 预取到新的结构 or PGO）
        
    3.  BTB替换算法（少见，业界一般使用LRU，PLRU）
        
    4.  BTB压缩（业界一般不做）
        
    5.  skewed BTB（每个way索引方式不同，打乱同block不同分支的位置）
        
4.  ahead pipeline/prediction（不清楚）
    
5.  two taken（arm旗舰核心标配，暂无论文支撑，一般设计为limit two taken，会限制第二个预测分支的类型或者会限制跳转地址）
    
6.  Trace Cache（可能苹果使用了这个）
    

前端大致分为上述方向，其他为偏工程化的实现，

分支预测建议先读TAGE源论文，之后可以读TAGE-SC-L，之后读TAGE: an engineering cookbook

BTB先了解常见结构，有类似综述的文章

指令预取直接上FDIP

其他可以少了解

### 后端

> **后端目前相对比较成熟，可供选择的方向较少**

1.  value prediction（更早打破数据依赖，业界有做）
    
2.  addr prediction （没听说过）
    
3.  sch（调度方法）
    
4.  checkpoint（如果做高性能checkpoint）
    
5.  regfile（一般研究如何降低功耗和面积，优化PPA）
    
6.  rob（目前业界应该都有ROB压缩）
    
7.  RVV（如何设计高性能RVV）
    
8.  reg early release（提前释放寄存器）
    

### 访存&缓存

1.  LSQ结构优化
    
2.  MDP（内存依赖预测）
    
3.  内存一致性（TSO,RVWMO...）
    
4.  Cache
    
    1.  way prediction（降功耗）
        
    2.  替换算法（太多了，目前业界L1D主流PLRU,LRU，L2 arm使用DRRIP）
        
    3.  预取（太多了，一般L1D主流stride，stream，tmp，L2 arm使用BOP）
        
        1.  spatial
            
        2.  temporal
            
        3.  exec based
            
        4.  PGO Based
            
        5.  预取调度
            
        6.  filter
            
    4.  缓存压缩（省面积，没听说有公司使用）
        
    5.  死块预测（没听说）
        
    6.  缓存一致性（MESI OR MOESI）
        
5.  NOC（目前业界常用CMN700，这个其实可以单开方向）
    

### 总结

目前CPU核内部可以做的其实只有分支预测，基于FDIP的指令预取，BTB，预取（最多的方向），替换（近几年替换也不多）

## 模拟器

CPU设计是一个庞大的工程，一个想法如果在RTL上落地会耗费大量的时间，所以需要一种快速评估的方法：模拟器

目前比较主流的有gem5[gem5/gem5: The official repository for the gem5 computer-system architecture simulator.](https://github.com/gem5/gem5)

champsim：[ChampSim/ChampSim: ChampSim is an open-source trace based simulator maintained at Texas A&M University and through the support of the computer architecture community.](https://github.com/ChampSim/ChampSim)

个人更推荐champsim，**有trace提供**，不用自己去打checkpoint，而且比gem5要简单很多，不过**建模粒度粗糙**

gem5整体代码很多，建模比较细致，**学习成本高**

可以自己在模拟器实现一些比较基础的算法or使用模拟器自带的算法来跑跑数据，更深入的理解微架构

对于分支预测，可以去观察MPKI，一个程序的MPKI很高有两种可能：

1.  分支预测器太烂，学习太差劲
    
2.  H2P（Hard to Predict）分支太多，分支预测器无法cover住
    

## 性能分析

可以看看intel的top down分析方法

## 逆向工程

可以看看苹果M1逆向cookbook