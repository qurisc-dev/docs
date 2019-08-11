Peripherals
========

QuSim用到的片外设备，包括SRAM、串口、GPIO。

SRAM
--------

我们将异步SRAM封装成一个标准的AXI Slave，数据位宽为64位，地址宽度20位，总大小8MiB，支持AXI协议规定的多种Burst传输方式（Fixed、Incr、Wrap）。

为了尽可能提高SRAM的效率，我们要求SRAM控制器工作在100MHz下，使得信号保持时间粒度下降到10ns。
进而，我们规定以下的状态机，并且根据以下的原则设置SRAM的CE、WE和OE等参数：

.. _sram_stm:
.. figure:: /figures/sram_stm.png
    :alt: SRAM Controller State Machine
	
- 片选永远有效，CE永远为0（需要放弃支持和Base SRAM共用总线的串口而使用直连串口）。
- 读操作时WE总是无效，写操作时OE总是无效。
- READ_0和READ_1时，OE有效。
- WRITE_0和WRITE_1时，WE有效；WRITE_2时，WE被拉高无效。

.. note::
   WE保持有效20ns是为了满足12ns的时序要求。
   
   违反12ns的时序约束可以把三个状态缩减到两个，但是该做法没有经过测试。为了保险，我们使用三个状态。
   
- SRAM所有的输入（地址、控制信号、输入数据、三态门开关）都直接由一个Flip-flop控制，防止输出信号毛刺（尤其是WE毛刺带来的错误上升沿，这可能是致命的）。
- 其余时候，无关的变量均设为无效。


SRAM控制器被封装成一个独立的IP核，方便用于Block Design和重用。

.. warning::
   SRAM控制器的Fixed和Wrap模式都未经过仔细测试，可能会包含未知的Bug。
   
   
串口
--------

.. warning::
   该串口控制器符合监控程序的默认要求，但是没有任何流控制机制，在输入得不到处理的时候可能会导致数据丢失。
   
   在一般情况下，推荐使用 `Xilinx AXI Uartile <https://www.xilinx.com/support/documentation/ip_documentation/axi_uartlite/v2_0/pg142-axi-uartlite.pdf>`_ ，
   但是需要根据该IP核的规范，手动调整操作系统或者监控程序从串口收发数据的方式。
   
我们简单地封装async_receiver和async_transmitter成为一个AXI Lite设备，具有数据和状态两个位置。从数据位置读取为读串口，向其写入为写串口；
状态寄存器只读，显示当前能否读写串口；软件必须保证，读写串口的时候状态寄存器对应的位有效。

串口收到的数据被写入一个FIFO里，由处理器读取；向串口写数据不设FIFO，直接由状态机管理写入。

串口控制器被封装成一个独立的IP核，方便用于Block Design和重用。

GPIO
--------

我们使用Xilinx AXI GPIO管理片上的LED等设备。


ROM
--------

.. warning::
   ROM还未实现。
   
我们使用一块Block RAM作为ROM。