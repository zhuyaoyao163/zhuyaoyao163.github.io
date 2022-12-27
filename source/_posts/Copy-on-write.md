---
title: Copy-on-write
date: 2022-06-14 11:33:16
tags:
---

## 简介
> 写入时复制（英语：Copy-on-write，简称COW）是一种计算机程序设计领域的优化策略。其核心思想是，如果有多个调用者（callers）同时请求相同资源（如内存或磁盘上的数据存储），他们会共同获取相同的指针指向相同的资源，直到某个调用者试图修改资源的内容时，系统才会真正复制一份专用副本（private copy）给该调用者，而其他调用者所见到的最初的资源仍然保持不变。这过程对其他的调用者都是透明的。此作法主要的优点是如果调用者没有修改该资源，就不会有副本（private copy）被创建，因此多个调用者只是读取操作时可以共享同一份资源。

## 应用
1. redis使用rdb方式进行持久化时，主线程fork出bgsave子进程，bgsave子进程实际上是复制了主线程的页表，这些页表，保存了主线程内存中数据的物理地址，这样以来bgsave进程可以根据这些页表中的数据生成rdb文件，再写入磁盘文件。如果此时主线程接受到写的命令，主线程就会用到写时复制机制。父子进程写时复制指的是，父子进程在刚开始的时候页表都是一样的（子进程从父进程复制过来的），如果父进程先修改了内存中的内容，那么父进程会重新申请一个内存页，然后将修改或者新增的内容写在这个内存页中，并且修改自己的页表映射地址。应该是遵循，谁修改谁复制的原则。

![img](https://zhuyaoyao-blog.oss-cn-hangzhou.aliyuncs.com/redis-copyonwrite.jpg)

2. java中的 **CopyOnWriteArrayList**是一个线程安全的`arraylist`,适用于读多写少的场景，每次写操作都会加锁，同时copy一个副本，array是用volatile修饰的，保证了线程的可见性，源码如下：
   
   ```
    private transient volatile Object[] array;

    public E get(int index) {
        return get(getArray(), index);
    }


    public E set(int index, E element) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            E oldValue = get(elements, index);

            if (oldValue != element) {
                int len = elements.length;
                Object[] newElements = Arrays.copyOf(elements, len);
                newElements[index] = element;
                setArray(newElements);
            } else {
                // Not quite a no-op; ensures volatile write semantics
                setArray(elements);
            }
            return oldValue;
        } finally {
            lock.unlock();
        }
    }
    ```
## 参考
- <https://zh.wikipedia.org/wiki/%E5%AF%AB%E5%85%A5%E6%99%82%E8%A4%87%E8%A3%BD>
- <https://time.geekbang.org/column/article/277373>