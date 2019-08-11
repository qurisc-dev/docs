TODO List
========

QuSim将被继续维护并进一步开发。以下为当前的坑。

L1 Cache
--------
.. _cache:
.. figure:: /figures/cache.png
    :alt: Cache

Cache被设计成8路组相联，Cache Line大小64Byte，每组含32项（大小可以削减），共占16KiB。在指定的FPGA上，恰可以放置两块Cache。

Cache命中时，只需要一个周期完成读操作，两个周期完成写操作；Cache未命中时，需要复杂的维护操作。

.. info::
   读取Cache的操作可以和读取Store Queue并行完成，因此读操作的失败不一定会立刻导致缓存重填。


Cache访问只需要基于下标寻址而不需要基于内容的比较，因此Cache的内容可以存储在一组Block RAM里，减少对Flip-flop的使用。
考虑到每次访问最多只有8字节，可以将8字节设为BRAM的数据宽度，而不需要每次读取整个Cache Line。

驱逐算法上使用基于二叉树的伪LRU算法，保证在每个分支上驱逐“最不常使用”的项。

为了保证效率，Cache是Write-back的。缓存一致性？

1. 当需要保证ICache和DCache一致的时候（sfence.vma或者ifence），等待DCache写回。
2. 使用缓存一致性协议。

TLB & Pagetable Walker
--------

ITLB, DTLB, JTLB
^^^^^^^^
TLB被计划设计成一个二级缓存结构：第一级为一个直接映射的缓存，第二级被设计成一个全相联的缓存。
ITLB/DTLB缺失时，会向JTLB请求TLB表项（保证了ITLB/DTLB的内容一定在JTLB中）。JTLB缺失时，会调用Page Table Walker访问主存储器重填表项或者返回异常。

Page Table Walker
^^^^^^^^
Page Table Walker会直接访问内存来获得页表表项。其设计应该为一个简单的状态机，并且作为一个AXI Master出现。
Page Table Walker不设置缓存。在DRAM情形下，这可能会带来严重的效率损失：可能需要一个接在DRAM前方的L2 Cache来提供加速。

FPU
--------

QuSim当前缺少浮点运算的能力。

Xilinx Floating Point IP核提供了浮点数运算的流水线，然而，该IP核不支持多种Rounding，这违反了RISC-V指令集对于浮点数运算指令的要求。

替代方案应该自行实现流水线，支持32位浮点运算和64位浮点运算，并且正确地抛出异常。

一些浮点数指令需要等待三个操作数，在当前的RS-FU框架下可能需要调整。

CSR
--------

QuSim缺少所有的CSR寄存器。

CSR寄存器指令本身不难实现，难点在于各个CSR的功能需要分别单独讨论，有些CSR甚至对应着一整个子系统（例如异常、中断、地址翻译等）。以下列举若干。

fcsr
^^^^^^^^

许多浮点指令可能产生异常。我们将浮点异常作为ROB的单独一栏，并且在指令提交时将浮点异常加入fcsr寄存器。
同时，FPU可以从fcsr中读取Rounding方式。


"Software" Engineering
--------

QuSim的开发严重依赖于Vivado IDE。Vivado给QuSim的开发带来了极大的便利，但也带来了许多的问题。

代码管理
^^^^^^^^

当前QuSim的代码仓库管理比较混乱，由于将整个Vivado工程加入了代码仓库，Git仓库大小飞速膨胀；同时，一些IP核未能及时加入代码仓库中。

我们需要整理QuSim的代码仓库，移除无必要的文件。
理想的做法是直到有必要时才创建Vivado项目。

构建测试
^^^^^^^^

当前，QuSim的测试工作主要通过人力借助波形图完成，没有自动测试机制或者单元测试。

我们需要一套自动构建和测试QuSim的工作流，以及补全我们需要的单元测试。

开发语言
^^^^^^^^

QuSim使用Verilog进行开发，而Verilog本身是弱类型的，缺少类型约束给开发和调试带来了极大的麻烦。

QuSim借助于Vivado的Block Design进行元件之间的组装。元件组装非常方便，但是这也导致开发极度依赖于Vivado本身。

IP核
^^^^^^^^

QuSim使用了大量Xilinx IP核，这些IP核可能是非开源的甚至是收费的，同时也阻止了进一步定制。

当前QuSim使用的最重要的Xilinx IP核是Xilinx AXI Interconnect。我们需要一个该IP核的开源替代品，或者切换到其它的总线协议。
对于其它的IP核，我们可以将其轻松替代。