# MicroInfer

一种适用于微处理器的深度学习推理框架，为多种不同算力的微处理器（目前规划为cortex-m4 、cortex-m7、cortex-a等）提供统一标准接口的一键式深度学习模型部署方案，让算法工程师在不深入底层的情况下，可以无差异化的验证模型的运行情况。

此框架使用独立的静态内存管理机制，可与本人设计的XidianOS操作系统配合使用，目前仅适用于MCU，暂不支持MMU功能。
https://github.com/Derekduke/XidianOS


## windows 本地调试方法
### windows10 安装make
参考：
https://www.cnblogs.com/jixiaohua/p/11724218.html

### 编译运行
cmd中输入“make”，生成bin.exe，输入“bin”运行

## 代码分析
### 基本要素：
### 通用层描述符
### 专用层描述符（包含通用层描述符）
tensor（用于描述某个内存中，存放数据的特征，比如CHW）
通用层所拥有的IO（IN或OUT两种）
通用层所拥有的BUF（其中包含用于中间计算的block和block的size）
IO所拥有的hook（用来指向下层的输入IO或上层的输出IO）
IO所拥有的block（用来存放IO中tensor描述的真实内存数据的指针）

### 运行逻辑：
1.创建各个层，主要工作是根据层的输入属性（如卷积核数量、大小等），初始化对应层的描述符和已经可确定的tensor
2.执行model_compile，主要工作是从头到位逐层遍历模型，确认每一层的输入、输出、计算，包括这三者所需要的tensor、内存块以及tensor和内存块的对应关系。
3.进行model_run

## 功能点推进：

- [x] 内存管理机制
- [x] 计算图可视化
- [x] 层实现（输入、卷积、池化、ReLU、全连接、输出） 
- [x] 模型构建（数据流指定及内存分配）
- [ ] 简单模型测试（手写数字）
- [ ] 分类模型测试（Cifar10物体分类）
- [ ] 中间层的跨平台后端支持统一框架
- [ ] 上层的用户输入处理接口
- [ ] 多种类模型的解析器（支持.h5和.tflite）
- [ ] 代码自动生成器（仅支持XidianOS的keil工程和Makefile工程即可）
- [ ] 算子优化和开关宏

## 创新点
### 1.简约高效的独立内存管理机制（已实现）

仅固定可使用内存块的数量，但按照尽可能多的复用“老”内存块的原则，根据长板效应对内存块的大小做动态调整，类似于贪心策略获得相对较优的内存分配方案。同时按照先计算后分配的原则，做到内存块在物理设备上的紧密相连，减少内存碎片化。

### 2.无差异的跨平台统一模型部署框架（部分实现）

已经在我的plugin_imx6ull仓库中实现应用，原本是贡献给RT-Thread操作系统在cortex-a处理器的一个插件，用于在基于RT_Thread的cortex-a7处理器上一键式的部署tensorflow lite模型。思路基本相同，暂时还未合并进这里。

### 3.多平台算子调优（部分实现）

已经在我的 tencentos-tiny-with-tflitemicro-and-iot 仓库中实现应用Cortex-m4/m7的DSP资源（实际上是替换CMSIS-NN算子）实现行人识别，暂未将实例整合进来；在Cortex-a上利用NEON指令做加速的Arm Compute Library库移植还有点问题，主要是C++和C的混编链接；还有针对普通算子的调优，如近似卷积算法、循环矩阵乘法等还未实现。

### 4.模型功能创新（待考虑）
基于这套推理框架做一些有创新型的模型或应用（先测试简单的已有模型）

### 5.和特定操作系统（XidianOS）做结合（待考虑）
对于单核芯片来说，性能上很难找到优化点，主要体现在适用的便捷性上。
1.根据OS的BSP工程去获得芯片型号，对应的去匹配加速方案。2.根据模型的计算图大小去半自动的调整操作系统的线程栈大小。3.可使用操作系统的标准文件系统API去运行深度学习模型。4.将2作为操作系统的一种特定驱动框架

## 参考开源项目
RT-Thread RTAK套件、华为MindSpore lite的codegen生成器

Keras API手册

NNOM框架、Caffe框架、Tensorflow Lite Micro框架