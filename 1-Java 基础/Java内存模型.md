Java**内存模型**定义了统一的内存模型，用于屏蔽不同的硬件架构，它定义了**在多线程环境中线程对共享内存中值的修改是否对其它线程立即可见**。

> 操作系统有自己的内存模型，c, c++是直接使用操作系统的内存模型，但是Java为了屏蔽各系统差异，定义了统一的内存模型，Java中不再关心每个CPU核心有自己的内存，然后共享主内存，而是把关注点转移到：每个线程都有自己的工作内存，所有线程共享主内存。

