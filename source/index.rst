.. QuSim documentation master file, created by
   sphinx-quickstart on Thu Aug  8 09:18:10 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to QuSim's documentation!
=================================

.. _qusim:
.. figure:: /figures/qusim_v0.1.png
    :alt: Qusim v0.1 Architecture

本文档的目的在于描述QuSim的硬件和软件设计。

1. QuSim的乱序执行CPU的设计。
2. QuSim的矩阵乘法加速器的设计。
3. 针对QuSim的编译器的设计。

.. toctree::
   :maxdepth: 2
   :caption: Contents:
   :numbered:

   sections/designpattern
   sections/if_pc
   sections/if_tlb
   sections/if_mem
   sections/if_queue
   sections/if_ctrl
   sections/dispatcher
   sections/reg_and_rename
   sections/rob
   sections/rs
   sections/lsu
   sections/peripheral
   sections/software
   sections/todo
   
Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
