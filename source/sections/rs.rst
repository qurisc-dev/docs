RS
========

.. _rs:
.. figure:: /figures/rs.png
    :alt: Reserve Station
	
保留站是Tomasulo算法的计算元件，所有的计算在这里实际完成。

保留站由若干仲裁器、寄存器和功能单元组成，CDB的数据由这里产生。

指令发射
--------
指令译码时，译码器根据指令得出指令需要的保留站类型；当指令发射进入保留站时，仲裁器从同种的保留站中挑选一个作为当前指令的保留站。
当保留站的所有操作数就绪时，计算自动开始执行；计算结束时，计算结果通过CDB仲裁器（一个Round-Robin的仲裁器）发射到CDB上。


功能单元
--------

功能单元是保留站的基本组成部分，负责具体的计算任务。
功能单元按照构造分为两类：基于状态机的计算单元和基于流水线的计算单元。

基于状态机的计算单元类似于多周期的CPU，同时只能计算一个任务；
保留站将数据发送给状态机，经过若干周期后，状态机将结果发送回保留站，并且等待发送到CDB。
一些简单的或者难以流水线化的计算单元，例如ALU和LSU采用的是这种模型。


.. _fu_behav_stm:
.. figure:: /figures/fu_behav_stm.png
    :alt: Function Unit Behaviour Model (STM)

另外一些计算单元，其需要的资源可能较多但是易于流水线化或者基于现有的流水线（例如Xilinx IP核），为每个保留站配备一份计算资源容易造成效率的浪费。
此时我们为一条流水线配备多个保留站，保留站必须经过仲裁才能发射进入流水线，流水线的结果再进入到保留站中。


.. _fu_behav_pipeline:
.. figure:: /figures/fu_behav_pipeline.png
    :alt: Function Unit Behaviour Model (Pipeline)

基于流水线的保留站的最大问题在于难于实现取消功能：一些时候，我们希望能够精确取消一些指令（例如重发射的情形）；
而简单地清空保留站可能会导致被清空的计算结果错误地流入该保留站。

- 修改流水线，允许流水线根据编号精确取消指令。
	- 当使用现成流水线时，难以修改现有流水线。
- 保留站按照异步流水线的方式，等待计算完毕后再取消。
	- 这会导致保留站的利用率降低。
- 保留站对节拍进行计数并且主动从流水线出口获取计算结果。
	- 这需要保留站知道自己的操作需要经过的流水级数，难以适用于复杂和可变流水线。
	
.. _fu_behav_pipeline_cntr:
.. figure:: /figures/fu_behav_pipeline_cntr.png
    :alt: Function Unit Behaviour Model (Pipeline with Counter)
	
- 为每次计算指定唯一的ID，而非只比较输出的计算单元编号。
	- 过短的ID可能会导致编号回卷，使得计算结果流入错误的保留站；过长的ID浪费资源。

.. note::
   事实上为了防止ID回卷，我们只要保证同一个ID在流水线中不会出现两次。

   考虑到每一个时钟周期ID一定会向下走一级流水线，ID的最多数目只需要比“该流水线计算需要的最多周期数”要多即可。
   当ID回卷导致某个小ID流入流水线时，上一个相同ID的计算任务一定已经流出。
	
	
ALU
--------
ALU是最基本的运算单元，负责RV64I整数指令集的运算。

ALU使用一个“单状态”的状态机完成，本质是一个组合逻辑，支持多种运算。

FPU
--------

.. warning::
   FPU尚未被实现。
   
   Xilinx Floating Point Core不支持多种Rounding，故无法用于实现RISC-V要求的浮点指令集。
   
FPU用于计算32位/64位浮点数。

由于浮点运算逻辑复杂、占用资源较多、种类较多，且易于流水化，FPU采用基于流水线的保留站实现。

LSU FU
--------

.. warning::
   当前只有一个共用的Load/Store Unit。这保证了强顺序但降低了效率。
   
LSU FU用于访存相关操作。

LSU FU被实现为一个计算地址、访TLB、访SQ、访存、对齐、等待PNR等状态的状态机，与LSU共同工作完成访存操作。

Store操作时，LSU FU把值写入到LSU的SQ中；Load操作时，LSU FU访问SQ和L1 Cache直接读取出需要的值。