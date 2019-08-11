IF-CONTROLLER
========

IF-CONTROLLER作为整个取值端（前端）的控制器，主要控制前端流水线各个流水级的取消操作。其本质是一个组合逻辑。

- 当发现来自ROB的跳转要求时，将IF-PC设为ROB要求的值，取消IF-TLB、IF-MEM，清空IF-QUEUE。
- 否则，当发现来自IF-QUEUE（Decoder）的分支预测跳转要求时，将IF-PC设为Decoder（分支预测器）要求的值，取消IF-TLB、IF-MEM。