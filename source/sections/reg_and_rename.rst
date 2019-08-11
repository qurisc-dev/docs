Registers & Rename Buffer
========

.. _reg_and_rename:
.. figure:: /figures/reg_and_rename.png
    :alt: Registers & Rename Buffer
	
寄存器堆和重命名缓冲用于维护“当前时刻即将发射的指令，面对的指令间寄存器数据依赖关系”。

Registers
--------
寄存器堆。在编号上，我们不区分通用寄存器和浮点寄存器，二者采用统一编号，在译码时才将二者区分开
（因此理论上后端可以执行“使用三个通用寄存器的浮点操作”，但是RISC-V指令的译码不会产生这种操作）。
寄存器堆接受来自ROB的写入，同时允许Dispatcher查询寄存器堆中的值。
由于Dispatcher的依赖关系查询规则，寄存器堆不需要从写到读的Forward。

RenameBuffer
--------
寄存器重命名站保存当前的寄存器重命名状况：0号寄存器永远不会被重命名，其余寄存器在Dispatcher发射的时候会被重命名。当提交时，只有当当前寄存器仍然属于本表项，重命名站中的别名才会被抹去。
为了减少延迟，RenameBuffer中保存了当前寄存器对应表项的依赖Mask。0号寄存器永远没有依赖。

为了减少从ROB中查询寄存器值的延迟，注意到我们并不总是需要查询ROB中所有的表项，而只需要查询“重命名缓冲中的有效项对应的值”。
因此，我们维护一份寄存器值的缓存，接收来自CDB的数据。

一个寄存器项的状态转移可以视为如下的状态机。

.. _rename_stm:
.. figure:: /figures/rename_stm.png
    :alt: Rename Buffer state machine.

重发射相关
--------

.. warning::
    读取预测和重发射尚未被实现。
	
RenameBuffer需要知道何时开始重新发射（start_reissue）：
当需要重新发射时，RenameBuffer会完全清空。这样做的目的是，保证重发射的指令之间的正确依赖关系（当重发射开始时，ROB相当于为空，）
