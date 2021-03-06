.. _link-chapter1:

第一章：RV64 裸机应用
==============================================

.. toctree::
   :hidden:
   :maxdepth: 4

   1app-ee-platform
   2remove-std
   3-1-mini-rt-usrland
   3-2-mini-rt-baremetal
   4understand-prog
   6practice

本章导读
------------------

大多数程序员的第一行代码都从 ``Hello, world!`` 开始，当我们满怀着好奇心在编辑器内键入仅仅数个字节，再经过几行命令编译（靠的是编译器）、运行（靠的是操作系统），终于在黑洞洞的终端窗口中看到期望中的结果的时候，一扇通往编程世界的大门已经打开。在本章第一节 :doc:`1app-ee-platform` 中，可以看到用Rust语言编写的非常简单的“Hello, world”应用程序。

不过我们能够隐约意识到编程工作能够如此方便简洁并不是理所当然的，实际上有着多层硬件和软件工具和支撑环境隐藏在它背后，才让我们不必付出那么多努力就能够创造出功能强大的应用程序。生成应用程序二进制执行代码所依赖的是以**编译器**为主的开发环境；运行应用程序执行码所依赖的是以**操作系统**为主的执行环境。

本章主要是设计和实现建立在裸机上的执行环境，从中对应用程序和它所依赖的执行环境有一个全面和深入的理解。

本章我们的目标仍然只是输出 ``Hello, world!`` ，但这一次，我们将离开舒适区，基于一个几乎空无一物的平台从零开始搭建我们自己的高楼大厦，
而不是仅仅通过一行语句就完成任务。所以，在接下来的内容中，我们将描述如何让 ``Hello, world!`` 应用程序逐步脱离对编译器、运行时和操作系统的依赖，最终在裸机上运行。这时，我们也可把这个能在裸机上运行的 ``Hello, world!`` 应用程序称为一种支持输出字符串的非常初级的“三叶虫”操作系统，它其实就是一个给应用提供各种服务（比如输出字符串）的库，方便了单一应用程序在裸机上的开发与运行。


实践体验
------------------
本章设计实现了一个支持显示字符串应用的简单操作系统--“三叶虫”操作系统。

获取本章代码：

.. code-block:: console

   $ git clone https://github.com/rcore-os/rCore-Tutorial-v3.git
   $ cd rCore-Tutorial-v3
   $ git checkout ch1

在 qemu 模拟器上运行本章代码，看看一个小应用程序是如何在QEMU模拟的计算机上运行的：

.. code-block:: console

   $ cd os
   $ make run

将 Maix 系列开发版连接到 PC，并在上面运行本章代码，看看一个小应用程序是如何在真实计算机上运行的：

.. code-block:: console

   $ cd os
   $ make run BOARD=k210

.. warning::

   **FIXME: 提供 wsl/macOS 等更多平台支持**

如果顺利的话，以 qemu 平台为例，将输出：

.. code-block::

   [rustsbi] Version 0.1.0
   .______       __    __      _______.___________.  _______..______   __
   |   _  \     |  |  |  |    /       |           | /       ||   _  \ |  |
   |  |_)  |    |  |  |  |   |   (----`---|  |----`|   (----`|  |_)  ||  |
   |      /     |  |  |  |    \   \       |  |      \   \    |   _  < |  |
   |  |\  \----.|  `--'  |.----)   |      |  |  .----)   |   |  |_)  ||  |
   | _| `._____| \______/ |_______/       |__|  |_______/    |______/ |__|

   [rustsbi] Platform: QEMU
   [rustsbi] misa: RV64ACDFIMSU
   [rustsbi] mideleg: 0x222
   [rustsbi] medeleg: 0xb109
   [rustsbi] Kernel entry: 0x80020000
   Hello, world!
   .text [0x80020000, 0x80022000)
   .rodata [0x80022000, 0x80023000)
   .data [0x80023000, 0x80023000)
   boot_stack [0x80023000, 0x80033000)
   .bss [0x80033000, 0x80033000)
   Panicked at src/main.rs:46 Shutdown machine!

除了 ``Hello, world!`` 之外还有一些额外的信息，最后关机。


.. note::

   RustSBI是啥？
   
   戳 :doc:`../appendix-c/index` 可以进一步了解RustSBI。