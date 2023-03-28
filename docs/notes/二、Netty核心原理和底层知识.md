---
title: 二、Netty核心原理和底层知识
permalink: /post/2-netty-core-principles-and-underlying-knowledge-1edhiu.html
date: 2023-03-24 13:41:55
meta:
  - name: keywords
    content: ''
  - name: description
    content: >-
      二netty核心原理和底层知识高并发io的底层原理io读写的基础原理为了避免用户进程直接操作内核保证内核安全操作系统将内存(虚拟内存)划分为两部分一部分是内核空间(kernelspace​)一部分是用户空间(userspace​)。在linux系统中内核模块运行在内核空间对应的进程处于内核态_而用户程序运行在用户空间对应的线程处于用户态。操作系统的核心是内核独立于普通的应用程序可以访问受保护的内核空间也有访问底层硬件设备的权限。内核空间总是驻留在内存中它是为操作系统的内核保留的。应用程序是不允许直接在内核
tags: []
categories: []
author:
  name: terwer
  link: https://github.com/XDUcoderT
---

# 二、Netty核心原理和底层知识

# 高并发IO的底层原理

## IO读写的基础原理

为了避免用户进程直接操作内核，保证内核安全，操作系统将内存(虚拟内存)划分为两部分，一部分是内核空间(`Kernel-Space`​)，一部分是用户空间(`User-space`​)。在Linux系统中，内核模块运行在内核空间，对应的进程处于内核态；而用户程序运行在用户空间，对应的线程处于用户态。

操作系统的核心是内核，独立于普通的应用程序,可以访问受保护的内核空间，也有访问底层硬件设备的权限。内核空间总是驻留在内存中，它是为操作系统的内核保留的。

应用程序是不允许直接在内核空间区域进行读写，也是不容许直接调用内核代码定义的函数的。每个应用程序进程都有一个单独的用户空间，对应的进程处于用户态，用户态进程不能访问内核空间中的数据，也不能直接调用内核函数的，因此要进行系统调用的时候，就要将进程切换到内核态才能进行。

内核态进程可以执行任意命令，调用系统的一切资源，而用户态进程只能执行简单的运算，不能直接调用系统资源，现在问题来了：用户态进程如何执行系统调用呢？答案为：用户态进程必须通过系统接口（ystem Call），才能向内核发出指令，完成调用系统资源之类的操作

用户程序进行IO的读写，依赖于底层的IO读写，基本上会用到底层的两大系统调用：sys_read & sys_write。虽然在不同的操作系统中，sys_read&sys_write两大系统调用的名称和形式可能不完全一样，但是他们的基本功能是一样的。

⚠️注意

> 应用程序进行IO的读写，依赖于底层IO的读写，基本上会用到底层的两大系统调用：sys_read & sys_write。
>
> 虽然在不同的操作系统中，sys_read & sys_write两大系统调用的名臣和形式可能不完全一样，但是他们的基本功能是一样的。

**操作系统层面的sys_read系统调用，并不是直接从物理设备把数据读取到应用的内存中；sys_write系统调用，也不是直接把数据写入到物理设备。**

上层应用无论是调用操作系统的sys_read，还是调用操作系统的sys_write，都会涉及缓冲区。

具体来说，上层应用通过操作系统的sys_read系统调用，是把数据从内核缓冲区复制到应用程序的进程缓冲区；

 上层应用通过操作系统的sys_write系统调用，是把数据从应用程序的进程缓冲区复制到操作系统内核缓冲区。   

 简单来说，应用程序的IO操作，实际上不是物理设备级别的读写，而是缓存的复制 sys_read&sys_write两大系统调用，都不负责数据在内核缓冲区和物理设备（如磁盘、网卡等）之间的交换。这项底层的读写交换操作，是由操作系统内核来完成的。所以，应用程序中的IO操作，无论是对Socket的IO操作，还是对文件的IO操作，都属于上层应用的开发，他们的在输入和输出维度上的执行流程，都是类似的，都是在内核缓冲区和进程缓冲区之间进行的数据交换。 

### 内核缓冲区和进程缓冲区

为什么设置那么多的缓冲区，导致读写过程这么麻烦呢？

缓冲区的目的，是为了减少频繁地与设备之间的物理交换。计算机的外部物理设备与内存和CPU相比，有着非常大的差距。外部设备的直接读写，涉及操作系统的中断。发生系统中断时，需要保存之前的进程数据和状态信息，而结束中断之后，还需要恢复之间的数据和状态等信息。为了减少底层系统的频繁中断所导致的时间损耗、性能损耗，于是出现了内核缓冲区。

有了内核缓冲区，操作系统会对内核缓冲区进行监控，等待缓冲区到达一定数量的时候，再进行IO设备的中断处理，集中执行物理设备的IO操作，通过这种机制来提升系统的性能。至于具体在什么时候执行系统中断(包括读中断、写中断)，则由操作系统的内核来决定，应用程序不需要关心。

> 上层应用程序使用sys_read系统调用时，仅仅把数据从内核缓冲区复制到上层应用的缓冲区(进程缓冲区)，上层应用使用sys_write系统调用时，仅仅把数据从应用的用户缓冲区复制到内核缓冲区中。

内核缓冲区与应用缓冲区在数量上也不同，在Linux系统中，操作系统内核只有一个内核缓冲区。而每个用户程序（进程）则有自己独立的缓冲区，叫做用户缓冲区或者进程缓冲区。Linux系统中的用户程序的IO读写程序，在大多数情况下，并没有进行实际的IO操作，而是在用户缓冲区和内核缓冲区之间直接进行数据的交换。区。Linux系统中的用户程序的IO读写程序，在大多数情况下，并没有进行实际的IO操作，而是在用户缓冲区和内核缓冲区之间直接进行数据的交换。

​![image](assets/image-20230325102639-ilmkx0y.png)​

**完整输入流程的两个阶段：**

1. 应用程序等待数据准备好
2. 从内核缓冲区向用户缓冲区复制数据

    如果是sys_read一个socket，那么以上两个阶段的具体处理流程如下：

    1. 第一个阶段，应用程序等待数据通过网络到达网卡，当锁等待的分组到达时，数据被操作系统复制到内核缓冲区
    2. 第二个阶段，内核将数据从内核缓冲区复制到应用到用户缓冲区

**再具体一点，如果是在C程序客户端和服务器端之间完成一次socket请求和响应（包括sys_read和sys_write）的数据交换，其完整的流程如下：**

1. 客户端发送请求：C程序客户端程序通过sys_write系统调用，将数据复制到内核缓冲区，Linux将内核缓冲区的请求数据通过客户端器的网卡发送出去。
2. 服务端系统接收数据：在服务端，这份请求数据会被服务端操作系统通过DMA硬件，从接收网卡中读取到服务端机器的内核缓冲区
3. 服务端C程序获取数据：服务端C程序通过sys_read系统调用，从Linux内核缓冲区复制数据，复制到C用户缓冲区。
4. 服务端业务处理：服务器在自己的用户空间中，完成客户端的请求锁对应的业务处理。
5. 服务端返回数据：服务器C程序完成处理后，构建好的响应数据，将这些数据从用户缓冲区写入内核缓冲区，这里用到的是sys_write系统调用，操作系统会负责将内核缓冲区数据发送出去。
6. 服务端系统发送程序：服务端Linux系统将内核缓冲区中的数据写入网卡，网卡通过底层的通信协议，会讲数据发送给目标客户端

## 五种重要的IO模型

首先弄清楚两个概念：

1. 阻塞和非阻塞：

    阻塞IO指的是需要内核IO彻底发生完成后，才返回到用户空间执行用户程序的操作指令，阻塞是指用户程序(发起IO请求的进程或者线程)的执行状态都是阻塞的。可以说传统的IO模型都是阻塞IO模型，并且在Java中，默认创建的socket都属于阻塞IO模型
2. 同步和异步：

    同步和异步可以看成是发起IO请求的两种方式。同步IO是指用户空间(进程或线程)是主动发起IO请求的一方，系统内核是被动接受方。异步IO则反过来，系统内核是主动发起IO请求的一方，用户空间是被动接受方。

### 同步阻塞IO(Blocking IO)

用户空间主动发起，需要等待内核IO操作彻底完成后，才返回到用户空间的IO操作，IO操作过程中，发起IO请求的用户进程(或者线程)处于阻塞状态。

在阻塞式IO模型中，Java应用程序从发起IO系统调用开始，一直到系统调用返回，在这段时间内，发起IO请求的Java进程（或者线程）是阻塞的。直到返回成功后，应用进程才能开始处理用户空间的缓存区数据。

​![image](assets/image-20230325110656-z21bgbh.png)​

举个例子，在Java中发起一个socket的sys_read读操作的系统调用，流程大致如下：

（1）从Java进行IO读后发起sys_read系统调用开始，用户线程（或者线程）就进入阻塞状态。

（2）当系统内核收到sys_read系统调用，就开始准备数据。一开始，数据可能还没有到达内核缓冲区（例如，还没有收到一个完整的socket数据包），这个时候内核就要等待。

（3）内核一直等到完整的数据到达，就会将数据从内核缓冲区复制到用户缓冲区（用户空间的内存），然后内核返回结果（例如返回复制到用户缓冲区中的字节数）。

（4）直到内核返回后，用户线程才会解除阻塞的状态，重新运行起来。

**阻塞IO的特点** 

在内核进行IO执行的两个阶段，发起IO请求的用户进程（或者线程）被阻塞了。

**阻塞IO的优点**

应用的程序开发非常简单；在阻塞等待数据期间，用户线程挂起，用户线程基本不会占用CPU资源。

**阻塞IO的缺点**

一般情况下，会为每个连接配备一个独立的线程，一个线程维护一个连接的IO操作。在并发量小的情况下，这样做没有什么问题。但是，当在高并发的应用场景下，需要大量的线程来维护大量的网络连接，内存、线程切换开销会非常巨大。在高并发应用场景中，阻塞IO模型是性能很低的，基本上是不可用的。

### 同步非阻塞IO(Non-Blocking IO)

非阻塞IO，指的是用户空间的程序不需要等待内核IO操作彻底完成，可以立即返回用户空间去执行后续的指令，即发起IO请求的用户进程(或者线程)处于非阻塞的状态，与此同时，内核会立即返回给用户一个IO的状态值。

> 阻塞和非阻塞的区别是什么呢？
>
> 阻塞是指用户进程(或者线程)一直在等待，而不能干别的事情
>
> 非阻塞是指用户进程(或者线程)拿到内核返回的状态值就返回自己的空间，可以去干别的事情。在Java中，非阻塞IO的socket套接字，要求被设置为NONBLOCK模式
>
> 这里所说的 NIO（同步非阻塞 IO）模型，并非 Java 编程中的 NIO（New IO）类库。

所为同步非阻塞NIO，指的是用户进程主动发起，不需要等待内核IO彻底操作完成之后，就能立即返回到用户空间的IO操作，IO操作过程中，发起IO请求的用户进程(或者线程)处于非阻塞状态。

在Linux系统下，socket连接默认是阻塞模式，可以通过设置将socket编程为非阻塞的模式，在NIO的模型中，应用程序一旦开始IO系统调用，会出现以下两种情况：

1. 在内核缓冲区中没有数据的情况下，系统调用会立即返回，返回一个调用失败的信息
2. 在内核缓冲区有数据的情况下，在数据的复制过程中系统是阻塞的，直到完成数据从内核缓冲区复制到用户缓冲区。复制完成后，系统调用返回成功，用户进程(或者线程)可以开始处理用户空间的缓存数据。

​![image](assets/image-20230326100314-42pybzd.png)​

举个例子：

发起一个非阻塞的socket的sys_read读操作的系统调用，流程如下：

1. 在内核数据没有准备好的阶段，用户线程发起IO请求时，立即返回。所以，为了读取到最终的数据，用户进程(或者线程)需要不断地发起IO请求调用。
2. 内核数据到达后，用户进程(或者线程)发起系统调用，用户进程(或者线程)阻塞。内核开始复制数据，它会将数据从内核缓冲区复制到用户缓冲区，然后内核返回结果(例如返回复制到的用户缓冲区的字节数)。
3. 用户进程(或者线程)在读数据时，没有数据会立即返回而不阻塞，用户空间需要经过多次的尝试，才能保证最终真正读到数据，而后继续执行。

**特点：**

应用程序的线程需要不断地进行IO系统调用，轮询数据是否已经准备好，如果没有准备好，就继续轮询，直到完成IO系统调用为止。

**优点：**

每次发起的IO系统调用，在内核等待数据的过程中可以立即返回。同步线程不会阻塞，实时性较好。

**缺点：**

不断地轮询内核，这将占用大量的CPU时间，效率低下。

> **说明**
>
> 同步非阻塞IO也可以简称为NIO，但是，它不是Java编程中的NIO，虽然它们的英文缩写一样，但是不能混淆。Java中的NIO(New IO)类库组件，所归属的不是基础IO模型中的NIO(None Blocking IO)模型，而是另外的一种模型，叫做IO多路复用模型

### IO多路复用

为了提高性能，操作系统引入了一类新的系统调用，专门用于查询IO文件描述符的（含socket连接）的就绪状态。在Linux系统中，新的系统调用为select/epoll系统调用。通过该系统调用，一个用户进程（或者线程）可以监视多个文件描述符，一旦某个描述符就绪（一般是内核缓冲区可读/可写），内核能够将文件描述符的就绪状态返回给用户进程（或者线程），用户空间可以根据文件描述符的就绪状态，进行相应的IO系统调用。

IO多路复用（IO Multiplexing）是高性能Reactor线程模型的基础IO模型，当然，此模型是建立在同步非阻塞的模型基础之上的升级版。

​![image](assets/image-20230326101834-78wmji5.png)

发起一个多路复用IO的sys_read读操作的系统调用，流程如下

1. 选择器注册。在这种模式中，首先，将需要sys_read操作的目标文件描述符(socket连接)提前注册到Linux的select/epoll选择器中，在Java中所对应的选择器类是Selector类。然后，才可以开启整个IO多路复用模型的轮询流程。
2. 就绪状态的轮询。通过选择器的查询方法，查询所有的提前注册过的目标文件描述符(socket连接)，的IO就绪状态，通过查询的系统调用，内核会返回一个就绪的sokect列表。当任何一个注册过的socket中的数据准备好或者就绪了，就是内核缓冲区中有数据了，内核就将该socket加入到就绪的列表中，并且返回就绪事件。
3. 用户线程获得了就绪状态的列表后，根据其中的socket连接，发起sys_read系统调用，用户线程阻塞。内核开始复制数据，将数据从内核缓冲区复制到用户缓冲区
4. 复制完成后，内核返回结果，用户线程才会解除阻塞的状态，用户线程读取到了数据，继续执行。

> 在用户线程进行IO就绪事件的轮询时，需要调用了选择器的select查询方法，发起查询的用户进行或者线程是阻塞的。当然，如果使用了查询方法的非阻塞的重载版本，发起查询的用户进程或者线程也不会阻塞，重载版本会立即返回。

**特点**：

IO多路复用模型的IO涉及两种系统调用，一种是IO操作的系统调用，另一种是select/epoll就绪查询系统调用。IO多路复用模型建立在操作系统的基础设施之上，即操作系统的内核必须能够提供多路分离的系统调用select/epoll。和NIO模型相似，多路复用IO也需要轮询。负责select/epoll状态查询调用的线程，需要不断地进行select/epoll轮询，查找出达到IO操作就绪的socket连接。

**优点：**

一个选择器查询线程，可以同时处理成千上万的网络连接，所以，用户程序不必创建大量的线程，也不必维护这些线程，从而大大减小了系统的开销。这是一个线程维护一个连接的阻塞IO模式相比，使用多路IO复用模型的最大优势

Java语言的NIO（New IO）组件所使用的，就是IO多路复用模型。

**缺点：**

本质上，select/epoll系统调用是阻塞式的，属于同步阻塞IO。都需要在读写事件就绪后，由系统调用本身负责进行读写，也就是说这个事件的查询过程是阻塞的

‍

### 信号驱动IO模型

在信号驱动IO模型中，用户线程通过向核心注册IO事件的回调函数，当某个事件发生时，内核使用信号（SIGIO）通知进程运行回调函数。然后进入IO操作的第二个阶段——执行阶段：用户线程会继续执行，在信号回调函数中调用IO读写操作来进行实际的IO请求操作。

信号驱动IO可以看成是一种异步IO，可以简单理解为系统进行用户函数的回调。只是，信号驱动IO的异步特性做的不彻底。为什么呢？ 信号驱动IO仅仅在IO事件的通知阶段是异步的，而在第二阶段，也就是在将数据从内核缓冲区复制到用户缓冲区这个过程，用户进程是阻塞的、同步的

信号驱动IO的基本流程是：用户进程通过系统调用，向内核注册SIGIO信号的owner进程和以及进程内的回调函数。内核IO事件发生后（比如内核缓冲区数据就位）后，通知用户程序，用户进程通过sys_read系统调用，将数据复制到用户空间，然后执行业务逻辑。

​![image](assets/image-20230326104357-i4y9www.png)​

发起一个异步IO的sys_read读操作的系统调用，流程如下：

1. 设置SIGIO信号的信号处理回调函数。

2. 设置该套接口的属主进程，使得套接字的IO事件发生时，系统能够将SIGIO信号传递给属主进程，也就是当前进程。

3. 开启该套接口的信号驱动I/O机制，通常通过使用fcntl方法的F_SETFL操作命令，使能（enable）套接字的 O_NONBLOCK非阻塞标志和O_ASYNC异步标志完成。

**优点：**

用户进程在等待数据时，不会被阻塞，能够提高用户进程的效率。具体来说：在信号驱动式I/O模型中，应用程序使用套接口进行信号驱动I/O，并安装一个信号处理函数，进程继续运行并不阻塞。

**缺点：**

1. 大量IO事件发生时，可以会由于处理不过来，导致信号队列溢出。
2. 对于处理UDP套接字来讲，对于信号驱动I/O是有用的。可是，对于TCP而言，由于致使SIGIO信号通知的条件为数众多，进行IO信号进一步区分的成本太高，信号驱动的I/O方式近乎无用。
3. 信号驱动IO可以看成是一种异步IO，可以简单理解为系统进行用户函数的回调。只是，信号驱动IO的异步特性做的不彻底。为什么呢？ 信号驱动IO仅仅在IO事件的通知阶段是异步的，而在第二阶段，也就是在将数据从内核缓冲区复制到用户缓冲区这个过程，用户进程是阻塞的、同步的

### 异步IO

异步IO，指的是用户空间与内核空间的调用方式大反转。用户空间的线程变成被动接受者，而内核空间成了主动调用者。在异步IO模型中，当用户线程收到通知时，数据已经被内核读取完毕，并放在了用户缓冲区内，内核在IO完成后通知用户线程直接使用即可。

​![image](assets/image-20230326104513-vcs1xsp.png)​

发起一个异步IO的sys_read操作流程如下：

1. 当用户线程发起了sys_read调用(可以理解为注册一个回调函数)，立即就可以开始去做其他的事，用户线程不阻塞。
2. 内核就开始了IO的第一个阶段：准备数据。等到数据准备好了，内核就会将数据从内核缓冲区复制到了用户缓冲区。
3. 内核会给用户线程发送一个信号(Signal)，或者调用用户线程注册的回调方法，告诉用户线程，sys_read系统调用已经完成了，数据已经读入到了用户缓冲区。
4. 用户线程读取用户缓冲区的数据，完成后续的业务操作。

特点：在内核等待数据和复制数据的两个阶段，用户线程都不是阻塞的。用户线程需要接收内核的IO操作完成的事件，或者用户线程需要注册一个IO操作完成的回调函数。正因为如此，异步IO有的时候也被称为信号驱动IO。

缺点：应用程序仅需要进行事件的注册与接收，其余的工作都留给了操作系统，也就是说，需要底层内核提供支持。

## Java NIO核心讲解

### Java NIO简介

在1.4版本之前，Java IO类库是阻塞式IO；从1.4版本开始，引进了新的异步IO库，被称为Java New IO类库，简称为Java NIO。Java NIO类库的目标，就是要让Java支持非阻塞IO，基于这个原因，更多的人喜欢称JavaNIO为非阻塞IO（Non-Block IO），称“老的”阻塞式Java IO为OIO（Old IO）。总体上说，NIO弥补了原来面向流的OIO同步阻塞的不足，它为标准Java代码提供了高速的、面向缓冲区的IO。

Java NIO类库包含以下三个核心组件：

* Channel（通道）
* Buffer（缓冲区）
* Selector（选择器）

​![image](assets/image-20230326135732-nbz98uc.png)​

Java NIO，属于第三种模型—— IO 多路复用模型。只不过，Java NIO组件提供了统一的应用开发API，为大家屏蔽了底层的操作系统的差异。

#### NIO与OIO的对比

在Java中，NIO与OIO的区别，主要体现在三个方面

1. OIO是面向流的，NIO是面向缓冲区的。

    面向流的OIO操作中，IO的read()操作总是以流式的方式顺序地从一个流中读取一个或多个字节，因此，我们不能随意改变读取指针的位置，也不能前后移动流中的数据。

    NIO中引入了(Channel)通道和Buffer缓冲区的概念。面向缓冲区的读取和写入，都是与Buffer进行交互。应用程序只需要从通道中读取数据到缓冲区，或者将数据从缓冲区写入到通道中。NIO不像OIO那样是顺序操作，可以随意地读取Buffer中任意位置的数据，可以随意修改Buffer中任意位置的数据。
2. OIO的操作是阻塞的，而NIO的操作是非阻塞的。

    OIO的操作是阻塞的，当一个线程调用read()或者write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入，该线程再次·期间不能干其他事情了。例如：我们调用一个read方法读取一个文件的内容，那么调用read的线程就会被阻塞住，直到read操作完成。

    NIO是如何做到非阻塞的呢？当我们调用read方法的时候，系统底层已经把数据准备好了。应用程序只需要从通道把数据复制到Buffer就行；如果没有数据，当前线程就可以去干别的事情，不需要进行阻塞等待。NIO非阻塞的根本原因是使用了通道和通道的IO多路复用技术。
3. OIO没有选择器的概念，而NIO有选择器的概念

    NIO技术的实现，是基于底层的IO多路复用技术实现的，比如在Windows中需要select多路复用组件的支持，在Linux系统中需要select/poll/epoll多路复用组件的支持。

#### 通道（Channel）

首先说一下Channel，国内大多翻译成“通道”。Channel的角色和OIO中的Stream(流)是差不多的。在OIO中，同一个网络连接会关联到两个流：一个输入流（Input Stream），另一个输出流（Output Stream），Java应用程序通过这两个流，不断地进行输入和输出的操作。

在NIO中，一个网络连接使用一个Channel（通道）表示，所有的NIO的IO操作都是通过连接通道完成的。一个通道类似于OIO中的两个流的结合体，既可以从通道读取数据，也可以向通道写入数据。 

​![image](assets/image-20230326134303-7t5ggxy.png)​

Channel和Stream的一个显著的不同是：Stream是单向的，譬如InputStream是单向的只读流，OutputStream是单向的只写流；而Channel是双向的，既可以用来进行读操作，又可以用来进行写操作。

NIO中的Channel的主要实现有：

1. FileChannel 用于文件IO操作
2. 2.DatagramChannel 用于UDP的IO操作
3. 3.SocketChannel 用于TCP的传输操作
4. 4.ServerSocketChannel 用于TCP连接监听操作

> 通道的本质是什么？

需要回到TCP/IP协议的四层模型的基础知识。具体如下：

​![image](assets/image-20230326140006-w6tjhkp.png)​

在TCP/IP协议四层模型的最底层为链路层。在最原始的物理链路时代，咱们数据传输的两头(发送方和接收方)会通过拉同轴电缆的方式，拉一条物理电缆(类似于后来更加高级的网线)，这条网线就代表一个双向的连接(connection)，通过这条电缆，双方可以完成数据的传输。数据传输一旦完成，需要把这条物理链路拆除。

> 当然，同轴电缆只是物理链接最为早期的版本，就像咱们软件开发一样，底层的传输链路层也会不断的迭代，不断的改进，不断的提升，变着法儿提升性能。所以，这种点对点的物理链路，很快升级为更加复杂的虚拟链路。只是对于咱们应用层的开发人员来说，虚拟机链路的知识更加的庞杂，不方便大家理解而已。

在操作系统的维度，为应用创建一个文件描述符

一个文件描述符（file descriptor）也有一个内核的数据结构和一个进程内的唯一编号来表示。然后，操作系统会把这个文件描述提供给应用层，应用层通过这个文件描述符（file descriptor）去对传输链路进行数据的读取和写入。

> 这里要把文件描述符和文件两个概念，稍加区分。文件这个概念，狭义的理解，就是磁盘上的文件。实际上，Linux 上的文件描述符，除了对磁盘文件做引用之外，还可以引用非磁盘文件

NIO中的TCP传输通道，实际上就是对底层的传输链路所对应的文件描述符（filedescriptor）的一种封装

#### 选择器（Selector）

> 回顾：什么是IO多路复用模型

IO多路复用模型就是一个进程/线程可以同时监视多个文件描述符(含socket连接)，一旦其中的一个或者多个文件描述符可读或者可写，该监听进程/线程能够进行IO事件的查询。

在Java应用层面，如何实现对多个文件描述符的监视呢？

需要用到一个非常重要的Java NIO组件——Selector 选择器。

Selector 选择器可以理解为一个IO事件的监听与查询器。通过选择器，一个线程可以查询多个通道的IO事件的就绪状态。

在介绍Selector选择器之前，首先介绍一下这个前置的概念：IO事件。什么是IO事件呢？表示通道某种IO操作已经就绪、或者说已经做好了准备。例如，如果一个新Channel链接建立成功了，就会在Server Socket Channel上发生一个IO事件，代表一个新连接一个准备好，这个IO事件叫做“接收就绪”事件。再例如，一个Channel通道如果有数据可读，就会发生一个IO事件，代表该连接数据已经准备好，这个IO事件叫做 “读就绪”事件。

Java NIO将NIO事件进行了简化，只定义了四个事件，这四种事件用SelectionKey的四个常量来表示：

 SelectionKey.OP_CONNECT

 SelectionKey.OP_ACCEPT

 SelectionKey.OP_READ

 SelectionKey.OP_WRITE

Selector的本质，就是去查询这些IO就绪事件，所以，它的名称就叫做 Selector查询者。从编程实现维度来说，IO多路复用编程的第一步，是把通道注册到选择器中，第二步则是通过选择器所提供的事件查询（select）方法，这些注册的通道是否有已经就绪的IO事件（例如可读、可写、网络连接完成等）。由于一个选择器只需要一个线程进行监控，所以，我们可以很简单地使用一个线程，通过选择器去管理多个连接通道

​![image](assets/image-20230326142615-47mb0fs.png)​

与OIO相比，NIO使用选择器的最大优势：系统开销小，系统不必为每一个网络连接（文件描述符）创建进程/线程，从而大大减小了系统的开销。

总之，通过Java NIO可以达到一个线程负责多个连接通道的IO处理，这是非常高效的。这种高效，恰恰就来自于Java的选择器组件Selector以及其底层的操作系统IO多路复用技术的支持。

#### 缓冲区

应用程序与通道（Channel）主要的交互，主要是进行数据的read读取和write写入。为了完成NIO的非阻塞读写操作，NIO为大家准备了第三个重要的组件——NIO Buffer（NIO缓冲区）。

Buffer顾名思义：缓冲区，实际上是一个容器，一个连续数组。Channel提供从文件、网络读取数据的渠道，但是读写的数据都必须经过Buffer。

​![image](assets/image-20230326142958-jf673fk.png)​

所谓通道的读取，就是将数据从通道读取到缓冲区中；所谓通道的写入，就是将数据从缓冲区中写入到通道中。缓冲区的使用，是面向流进行读写操作的OIO所没有的，也是NIO非阻塞的重要前提和基础之一。

### 详解Buffer类及其属性

NIO的Buffer缓冲区本质上就是一个内存快，既可以写入数据，也可以从中读取数据。Java NIO中代表缓冲区的Buffer类是一个抽象类，位于java.nio包中。

IO的Buffer的内部是一个内存块（数组），此类与普通的内存块（Java数组）不同的是：NIO Buffer对象，提供了一组比较有效的方法，用来进行写入和读取的交替访问。

> Bufffer类是一个非线程安全类

#### Buffer种类

Buffer类是一个抽象类，对应于Java的主要数据类型，在NIO中有8种缓冲区类，分别如下：

ByteBuffer、CharBuffer、DoubleBuffer、FloatBuffer、IntBuffer、LongBuffer、ShortBuffer、MappedByteBuffer。

前7种Buffer类型，覆盖了能在IO中传输的所有的Java基本数据类型。第8种类型MappedByteBuffer是专门用于内存映射的一种ByteBuffer类型。不同的Buffer子类，其能操作的数据类型能够通过名称进行判断，比如IntBuffer只能操作Integer类型的对象。实际上，使用最多的还是ByteBuffer二进制字节缓冲区类型。

#### Buffer类的重要属性

Buffer的子类会拥有一块内存，作为数据的可读写区，但是读写缓冲区并没有定义在Buffer基类，而是定义在具体的子类中。如ByteBuffer字类就拥有一个byte[]类型的数组成员`​ final byte[] hb`​，作为自己的读写缓冲区，数组的元素类型与Buffer字类的操作类型一致

为了记录读写的状态和位置，Buffer类额外提供了一些重要的属性，其中有以下三个重要的成员属性：

1. capacity
2. position
3. limit
4. mark

​![image](assets/image-20230326144345-c1d782v.png)​

##### capacity属性

Buffer类的capacity属性，表示内部容量的大小。一旦写入的对象数量超过了capacity容量，缓冲区就满了，不能再写入了。

Buffer类的capacity属性一旦初始化，就不能再改变。原因是什么呢？Buffer类的对象在初始化时，会按照capacity分配内部数组的内存，在数组内存分配好之后，它的大小当然就不能改变了。

前面讲到，Buffer类是一个抽象类，Java不能直接用来新建对象。在具体使用的时候，必须使用Buffer的某个子类，例如DoubleBuffer子类，该子类能写入的数据类型是double类型，如果在创建实例时其capacity是100，那么我们最多可以写入100个double类型的数据。

> capacity容量并不是指内部的内存快byte[]数组的字节数量，而是指能写入的数据对象的最大限制数量

##### position属性

Buffer类的position属性，表示当前的位置。position属性的值与缓冲区的读写模式有关。在不同的模式下，position属性值的含义是不同的，在缓冲区进行读写的模式改变时，position值会进行相应的调整。

**在写入模式下，position的值变化规则如下：**

1. 在刚进入到写入模式时，position值为0，表示当前的写入位置为从头开始。
2. 每当一个数据写到缓冲区之后，position会向后移动到下一个可写的位置。
3. 初始的position值为0，最大可写值为limit–1。当position值达到limit时，缓冲区就已经无空间可写了。

**在读模式下，position的值变化规则如下：**

1. 当缓冲区刚开始进入到读取模式时，position会被重置为0。

2. 当从缓冲区读取时，也是从position位置开始读。读取数据后，position向前移动到下一个可读的位置。

3. 在读模式下，limit表示可以读上限。position的最大值，为最大可读上限limit，当position达到limit时，表明缓冲区已经无数据可读。

Buffer的读写模式具体如何切换呢？当新建了一个缓冲区实例时，缓冲区处于写入模式，这时是可以写数据的。在数据写入完成后，如果要从缓冲区读取数据，这就要进行模式的切换，可以使用（即调用）flip翻转方法，将缓冲区变成读取模式。在从写入模式到读取模式的flip翻转过程中，position和limit属性值会进行调整，具体的规则是：

1. limit属性被设置成写入模式时的position值，表示可以读取的最大数据位置；

2. position由原来的写入位置，变成新的可读位置，也就是0，表示可以从头开始读。

##### limit属性

Buffer类的limit属性，表示可以写入或者读取的最大上限，其属性值的具体含义，也与缓冲区的读写模式有关，在不同的模式下，limit的值的含义是不同的，具体分为以下两种情况：

1. 在写入模式下，limit属性值的含义为可以写入的数据最大上限。在刚进入到写入模式时，limit的值会被设置成缓冲区的capacity容量值，表示可以一直将缓冲区的容量写满。

2. 在读取模式下，limit的值含义为最多能从缓冲区中读取到多少数据。

一般来说，在进行缓冲区操作时，是先写入然后再读取的。当缓冲区写入完成后，就可以开始从Buffer读取数据，可以使用flip翻转方法，这时，limit的值也会进行调整。具体如何调整呢？将写入模式下的position值，设置成读取模式下的limit值，也就是说，将之前写入的最大数量，作为可以读取的上限值。Buffer在flip翻转时的属性值调整，主要涉及position、limit两个属性，但是这种调整比较微妙，不是太好理解

**下面是一个简单例子：**

首先，创建缓冲区。新创建的缓冲区处于写入模式，其position值为0，limit值为最大容量capacity。然后，向缓冲区写数据。每写入一个数据，position向后面移动一个位置，也就是position的值加1。这里假定写入了5个数，当写入完成后，position的值为5。最后，使用flip方法将缓冲区切换到读模式。limit的值，先会被设置成写入模式时的position值，所以新的limit值是5，表示可以读取的最大上限是5。之后调整position值，新的position会被重置为0，表示可以从0开始读。缓冲区切换到读模式后，就可以从缓冲区读取数据了，一直到缓冲区的数据读取完毕。

除了以上capacity（容量）、position（读写位置）、limit（读写的限制）三个重要属性之外，Buffer还有一个比较重要的标记属性：mark（标记）属性。

mark属性的大致作用为：读位置或者写位置的一个备份，供后续恢复时使用。在缓冲区操作（读或者写）的过程当中，可以将当前的position的值，临时存入mark属性中；需要恢复的时候，可以再从mark中取出之前的值，恢复到position属性中，然后，后续可以重新从position位置开始处理（读取或者写入）。

**总结：**

|属性|说明|
| ----------| ---------------------------------------------------------------------------------------------------------|
|capacity|容量，即剋容纳的最大数量；在缓冲区创建时设置，不可改变。|
|limit|上限，缓冲区当前的数据量|
|position|位置，缓冲区中下一个要被读或写的元素的索引|
|mark|调用mark()方法来设置mark = position，再调用reset()可以让position恢复到mark标记的位置，即position = mark|

#### Buffer类的重要方法

##### allocate方法创建一个Buffer类的实例对象

​![image](assets/image-20230326151458-spywiqe.png)​

```java
public class testBuffer {
    static IntBuffer intBuffer = null;
    public static void main(String[] args) {
        intBuffer = IntBuffer.allocate(10);
        System.out.println("position = " + intBuffer.position());
        System.out.println("limit = " + intBuffer.limit());
        System.out.println("capacity = " + intBuffer.capacity());
    }
}
```

```java
position = 0
limit = 10
capacity = 10
```

##### put写入到缓冲区

![image](assets/image-20230326151703-c560rvu.png)​

```java
public class testBuffer {
    static IntBuffer intBuffer = null;
    public static void main(String[] args) {
        intBuffer = IntBuffer.allocate(10);
        for(int i = 0;i < 5; i++){
            intBuffer.put(i);
        }
        System.out.println("position = " + intBuffer.position());
        System.out.println("limit = " + intBuffer.limit());
        System.out.println("capacity = " + intBuffer.capacity());
    }
}
```

```java
position = 5
limit = 10
capacity = 10
```

##### flip翻转

向缓冲区写入数据之后，是否可以直接从缓冲区中读取数据呢？呵呵，不能。为什么呢？这时缓冲区还处于写模式，如果需要读取数据，还需要将缓冲区转换成读模式。flip()翻转方法是Buffer类提供的一个模式转变的重要方法，它的作用就是将写入模式翻转成读取模式。

​![image](assets/image-20230326152022-9apx659.png)​

```java
public class testBuffer {
    static IntBuffer intBuffer = null;
    public static void main(String[] args) {
        intBuffer = IntBuffer.allocate(10);
        for(int i = 0;i < 5; i++){
            intBuffer.put(i);
        }
        intBuffer.flip();
        System.out.println("position = " + intBuffer.position());
        System.out.println("limit = " + intBuffer.limit());
        System.out.println("capacity = " + intBuffer.capacity());
    }
}
```

```java
position = 0
limit = 5
capacity = 10
```

 对flip()方法的从写入到读取转换的规则，再一次详细的介绍如下：

1. 首先，设置可读上限limit的属性值。将写入模式下的缓冲区中内容的最后写入位置position值，作为读取模式下的limit上限值。

2. 其次，把读的起始位置position的值设为0，表示从头开始读。

3. 最后，清除之前的mark标记，因为mark保存的是写入模式下的临时位置，发生模式翻转后，如果继续使用旧的mark标记，会造成位置混乱。

​![image](assets/image-20230326153115-ertqy1q.png)​

##### get从缓冲区读取

```java
public class testBuffer {
    static IntBuffer intBuffer = null;
    public static void main(String[] args) {
        intBuffer = IntBuffer.allocate(10);
        for(int i = 0;i < 5; i++){
            intBuffer.put(i);
        }
        intBuffer.flip();
        System.out.println("position = " + intBuffer.position());
        System.out.println("limit = " + intBuffer.limit());
        System.out.println("capacity = " + intBuffer.capacity());
        for(int i = 0;i < 3;i++){
            int num = intBuffer.get();
            System.out.println("num = " + num);
        }
        System.out.println("position = " + intBuffer.position());
        System.out.println("limit = " + intBuffer.limit());
        System.out.println("capacity = " + intBuffer.capacity());
    }
}
```

```java
position = 0
limit = 5
capacity = 10
num = 0
num = 1
num = 2
position = 3
limit = 5
capacity = 10
```

##### rewind倒带

已经读完的数据，如果需要再读一遍，可以调用rewind()方法。rewind()也叫倒带，就像播放磁带一样倒回去，再重新播放。

‍

```java
public class testBuffer {
    static IntBuffer intBuffer = null;
    public static void main(String[] args) {
        intBuffer = IntBuffer.allocate(10);
        for(int i = 0;i < 5; i++){
            intBuffer.put(i);
        }
        intBuffer.flip();
        System.out.println("position = " + intBuffer.position());
        System.out.println("limit = " + intBuffer.limit());
        System.out.println("capacity = " + intBuffer.capacity());
        for(int i = 0;i < 3;i++){
            int num = intBuffer.get();
            System.out.println("num = " + num);
        }
        System.out.println("position = " + intBuffer.position());
        System.out.println("limit = " + intBuffer.limit());
        System.out.println("capacity = " + intBuffer.capacity());
        intBuffer.rewind();
        System.out.println("position = " + intBuffer.position());
        System.out.println("limit = " + intBuffer.limit());
        System.out.println("capacity = " + intBuffer.capacity());
    }
}
```

```java
position = 0
limit = 5
capacity = 10
num = 0
num = 1
num = 2
position = 3
limit = 5
capacity = 10
position = 0
limit = 5
capacity = 10
```

##### mark和reset

```java
public class testBuffer {
    static IntBuffer intBuffer = null;
    public static void main(String[] args) {
        intBuffer = IntBuffer.allocate(10);
        for(int i = 0;i < 5; i++){
            intBuffer.put(i);
        }
        intBuffer.flip();
        System.out.println("position = " + intBuffer.position());
        System.out.println("limit = " + intBuffer.limit());
        System.out.println("capacity = " + intBuffer.capacity());
        for(int i = 0;i < 3;i++){
            int num = intBuffer.get();
            if(i == 1){
                intBuffer.mark();
            }
            System.out.println("num = " + num);
        }
        System.out.println("position = " + intBuffer.position());
        System.out.println("limit = " + intBuffer.limit());
        System.out.println("capacity = " + intBuffer.capacity());
        intBuffer.reset();
        System.out.println("position = " + intBuffer.position());
        System.out.println("limit = " + intBuffer.limit());
        System.out.println("capacity = " + intBuffer.capacity());
    }
}
```

```java
position = 0
limit = 5
capacity = 10
num = 0
num = 1
num = 2
position = 3
limit = 5
capacity = 10
position = 2
limit = 5
capacity = 10
```

##### clear()清空缓冲区

在读取模式下，调用clear方法将缓冲区切换为写入模式。此方法的作用：会将position清零；limit设置为capacity最大容量值，可以一直写入，直到缓冲区写满。

```java
public class testBuffer {
    static IntBuffer intBuffer = null;
    public static void main(String[] args) {
        intBuffer = IntBuffer.allocate(10);
        for(int i = 0;i < 5; i++){
            intBuffer.put(i);
        }
        intBuffer.flip();
        System.out.println("position = " + intBuffer.position());
        System.out.println("limit = " + intBuffer.limit());
        System.out.println("capacity = " + intBuffer.capacity());
        for(int i = 0;i < 3;i++){
            int num = intBuffer.get();
            System.out.println("num = " + num);
        }
        intBuffer.clear();
        System.out.println("position = " + intBuffer.position());
        System.out.println("limit = " + intBuffer.limit());
        System.out.println("capacity = " + intBuffer.capacity());

    }
}
```

```java
position = 0
limit = 5
capacity = 10
num = 0
num = 1
num = 2
position = 0
limit = 10
capacity = 10
```

### 详解NIO Channel（通道）类

前面提到，Java NIO中，一个socket连接使用一个Channel（通道）来表示。然而，从更广泛的层面来说，一个通道封装了一个底层的文件描述符，例如硬件设备、文件、网络连接等。所以，与文件描述符相对应，Java NIO的通道分为很多类型，但是Java的通道更加的细化，例如：对应到不同的网络传输模型，在Java中都有不同的NIO Channel（通道）相对应。

#### Channel（通道）的主要类型

这里不对Java NIO全部通道类型进行过多的描述，仅仅聚焦于介绍其中最为重要的四种

Channel（通道）实现：FileChannel、SocketChannel、ServerSocketChannel、DatagramChannel。

对于以上四种通道，说明如下：

1. FileChannel文件通道，用于文件的数据读写；
2. SocketChannel套接字通道，用于Socket套接字TCP连接的数据读写；
3. ServerSocketChannel服务器套接字通道（或服务器监听通道），允许我们监听TCP连接请求，为每个监听到的请求，创建一个SocketChannel套接字通道；
4. DatagramChannel数据报通道，用于UDP协议的数据读写。

这四种通道，覆盖了文件IO、TCP网络、UDP IO三类基础IO操作。下面从通道的获取、读取、写入、关闭四个重要的操作入手，对四种通道进行简单的介绍。

##### FileChannel文件通道

FileChannel是专门操作文件的通道。通过FileChannel，既可以从一个文件中读取数据，也可以将数据写入文件中。

> 注意⚠️：FileChannel为阻塞模式，不能设置为非阻塞模式

1. 获取FileChannel通道

    可以通过一个文件的输入流、输出流获取FileChannel文件通道，如：

    ```java
    //创建一个文件输入流
    FileInputStream fis = new FileInputStream(srcFile);
    //获取文件流的通道
    FileChannel inChannel = fis.getChannel();
    //创建一个文件输出流
    FileOutputStream fos = new FileOutputStream(destFile);
    //获取文件流的通道
    FileChannel outchannel = fos.getChannel();
    ```

    ‍

    或者通过`RandomAccessFile`​文件随机访问类，获取FileChannel文件通道实例

    ```java
    // 创建 RandomAccessFile 随机访问对象 
    RandomAccessFile rFile = new RandomAccessFile("filename.txt"，"rw");
     //获取文件流的通道（可读可写） 
    FileChannel channel = rFile.getChannel();
    ```

2. 读取FileChannel通道

    在大部分应用场景，从通道读取数据都会调用通道的int read方法(ByteBufferbuf)方法，它从通道读取数据写入到ByteBuffer缓冲区，并且返回读取到的数据量。

    ```java
    RandomAccessFile aFile = new RandomAccessFile(fileName, "rw");

    //获取通道（可读可写）

    FileChannel channel=aFile.getChannel();

    //获取一个字节缓冲区

    ByteBuffer buf = ByteBuffer.allocate(CAPACITY);

    int length = -1;

    //调用通道的 read 方法，读取数据并买入字节类型的缓冲区

    while ((length = channel.read(buf)) != -1) {

    //……省略 buf 中的数据处理

    }
    ```

    > 注意⚠️：以上代码虽然是读取通道的数据，但是相对于缓冲区来讲是写入模式
    >

3. 写入FileChannel通道

    写入数据到通道，在大部分应用场景，都会调用通道的write(ByteBuffer)方法，此方法的参数是一个ByteBuffer缓冲区实例，是待写数据的来源

    write(ByteBuffer)方法的作用，是从ByteBuffer缓冲区中读取数据，然后写入到通道自身，而返回值是写入成功的字节数。

    ```java
    //如果 buf 处于写入模式（如刚写完数据），需要 flip 翻转 buf，使其变成读取模式 
    buf.flip(); 
    int outlength = 0; 
    //调用 write 方法，将 buf 的数据写入通道 
    while ((outlength = outchannel.write(buf)) != 0) 
    { 
    	System.out.println("写入的字节数：" + outlength); 
    }
    ```

    在以上的outchannel.write(buf)调用中，对于入参buf实例来说，需要从其中读取数据写入到outchannel通道中，所以入参buf必须处于读取模式，不能处于写入模式。

4. 关闭通道

    当通道使用完成后，必须将其关闭。关闭非常简单，调用close( )方法即可。

    ```java
    //关闭通道
    channel.close( );
    ```

5. 强制刷新到磁盘

    在将缓冲区写入通道时，由于性能原因，操作系统不可能每次都实时豆浆写入数据落地(或刷新)到磁盘，完成最终的数据保存。

    如果在将缓冲数据写入通道时，需要保证数据能落地写入到磁盘，可以在写入后调用后调用一下FileChannel的force方法

    ```java
    //强制刷新到磁盘 
    channel.force(true);
    ```

**实例：**

```java
public class testFileChannel {
    public static void main(String[] args) {
        File srcFile = new File("/Users/xducodert/Desktop/io-study/src/main/resources/Buffer/123.txt");
        File destFile = new File("/Users/xducodert/Desktop/io-study/src/main/resources/Buffer/234.txt");
        try {
            //如果目标文件不存在，则新建
            if (!destFile.exists()) {
                destFile.createNewFile();
            }
            FileChannel inChannel = null;
            FileChannel outchannel = null;
            try {
                RandomAccessFile fis = new RandomAccessFile(srcFile,"rw");
                RandomAccessFile fos = new RandomAccessFile(destFile,"rw");
                inChannel = fis.getChannel();
                outchannel = fos.getChannel();

                int length = -1;
                ByteBuffer buf = ByteBuffer.allocateDirect(1024);
                //从输入通道读取到buf
                while ((length = inChannel.read(buf)) != -1) {
                    //翻转buf,变成成读模式
                    buf.flip();
                    int outlength = 0;
                    //将buf写入到输出的通道
                    while ((outlength = outchannel.write(buf)) != 0) {
                        System.out.println("写入字节数：" + outlength);
                    }
                    //清除buf,变成写入模式
                    buf.clear();
                }
                //强制刷新磁盘
                outchannel.force(true);
            } finally {
                outchannel.close();
                inChannel.close();
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

```plaintext
写入字节数：97
```

##### SocketChannel套接字通道

在NIO中，涉及网络连接的通道有两个：一个是SocketChannel负责连接的数据传输，另一个是ServerSocketChannel负责连接的监听。其中，NIO中的SocketChannel传输通道，与OIO中的Socket类对应；NIO中的ServerSocketChannel监听通道，对应于OIO中的ServerSocket类。

ServerSocketChannel仅仅应用于服务器端，而SocketChannel则同时处于服务器端和客户端，所以，对应于一个连接，两端都有一个负责传输的SocketChannel传输通道。

> 无论是ServerSocketChannel，还是SocketChannel，都支持阻塞和非阻塞两种模式。如何
>
> 进行模式的设置呢？
>
> 1. socketChannel.configureBlocking(false)设置为非阻塞模式
>
> 2. socketChannel.configureBlocking(true)设置为阻塞模式

在阻塞模式下，SocketChannel通道的connect连接、read读、write写操作，都是同步的和阻塞式的，在效率上与Java旧的OIO的面向流的阻塞式读写操作相同。因此，在这里不介绍阻塞模式下的通道的具体操作。在非阻塞模式下，通道的操作是异步、高效率的，这也是相对于传统的OIO的优势所在。

下面仅仅详细介绍在非阻塞模式下通道的打开、读写和关闭操作等操作。

1. 获取SocketChannel传输通道

    在客户端，首先通过SocketChannel静态方法open()获取一个套接字传输通道；然后，将socket套接字设置为非阻塞模式，最后，通过connect()实例方法，对服务器的IP和端口发起连接

    ```java
     FileChannel fileChannel = new FileInputStream(file).getChannel();
     //通过SocketChannel的静态方法open()获取一个套接字传输通道
     SocketChannel socketChannel = SocketChannel.open();
     socketChannel.setOption(StandardSocketOptions.TCP_NODELAY, true);
    //对服务器的IP和端口发起连接
    socketChannel.socket().connect(
           	new InetSocketAddress(NioDemoConfig.SOCKET_SERVER_IP
    	, NioDemoConfig.SOCKET_SERVER_PORT));
    	socketChannel.configureBlocking(false);
    ```

    在服务器端，当连接事件到来时，ServerSocketChannel能成功的查询出这个新连接事件，并且通过accet()方法能够获取新连接的套接字通道。

    ```java
    //新连接事件到来，首先通过事件，获取服务器监听通道 
    ServerSocketChannel server = (ServerSocketChannel) key.channel(); 
    //获取新连接的套接字通道 
    SocketChannel socketChannel = server.accept(); 
    //设置为非阻塞模式 
    socketChannel.configureBlocking(false);
    ```

2. 读取SocketChannel传输通道

    当SocketChannel传输通道可读时，可以从SocketChannel读取数据，具体方法与前面的文件通道读取方法是相同的。调用read方法，将数据读入缓冲区ByteBuffer。

    ```java
    ByteBufferbuf = ByteBuffer.allocate(1024);

    int bytesRead = socketChannel.read(buf);
    ```

    在读取时，因为是异步的，因此我们必须检查read的返回值，以便判断当前是否读取到了数据。read()方法的返回值是读取的字节数，如果返回-1，那么表示读取到对方的输出结束标志，对方已经输出结束，准备关闭连接。实际上，通过read方法读数据，本身是很简单的，比较困难的是，在非阻塞模式下，如何知道通道何时是可读的呢？这就需要用到NIO的新组件——Selector通道选择器，稍后介绍。

3. 写入到SocketChannel通道

    和前面的把数据写入到FileChannel文件通道一样，大部分应用场景都会调用通道的int write（ByteBufferbuf）方法。

    ```java
    //写入前需要读取缓冲区，

    //要求 ByteBuffer 是读取模式

    buffer.flip();

    socketChannel.write(buffer);
    ```

4. 关闭SocketChannel通道

    在关闭SocketChannel传输通道前，如果传输通道用来写入数据，则建议调用一次shutdownOutput()终止输出方法，向对方发送一个输出的结束标志（-1）。然后调用socketChannel.close()方法，关闭套接字连接。

    ```java
    //调用终止输出方法，向对方发送一个输出的结束标志

    socketChannel.shutdownOutput();

    //关闭套接字连接

    IOUtil.closeQuietly(socketChannel);
    ```

### 详解NIO Selector选择器

#### 选择器及其注册

选择器的使命就是完成IO的多路复用，其主要主要工作是通道的注册、监听、事件查询。一个通道代表一条连接通路，通过选择器可以同时监控多个通道的IO(输入输出)状况。

选择器与通道的关系，是监控与被监控的关系。

选择器可以通过select，查询出所监控的通道已经发生了哪些IO事件，包括读写就绪的IO操作事件。

在NIO编程中，一般是一个单线程处理一个选择器，一个选择器可以监控很多通道。所以，通过选择器，一个单线程可以处理很多的通道，可以大量的减少线程上下文切换的开销。

通道和选择器之间的关联，通过register（注册）的方式完成。调用通道的Channel.register（Selector sel，int ops）方法，可以将通道实例注册到一个选择器中。register方法有两个参数：第一个参数，指定通道注册到的选择器实例；第二个参数，指定选择器要监控的IO事件类型。

可供选择器监控的IO事件类型，包括以下四种：

1. 可读：SelectionKey.OP_READ

2. 可读：SelectionKey.OP_WRITE

3. 可读：SelectionKey.OP_CONNECT

4. 可读：SelectionKey.OP_ACCEPT

以上的事件类型常量定义在SelectionKey类中。如果选择器要监控通道的多种事件，可以用“按位或”运算符来实现。例如，同时监控可读和可写IO事件：

```java
//监控通道的多种事件，用“按位或”运算符来实现

int key = SelectionKey.OP_READ | SelectionKey.OP_WRITE
```

#### IO事件

IO事件是指IO操作的就绪状态，表示通道具备执行某个IO的条件

比如：

1.  某个SocketChannel传输通道，如果完成了和对端的三次握手过程，则会发生“连接就绪”

    （OP_CONNECT）的事件
2.  某个ServerSocketChannel服务器连接监听通道，在监听到一个新连接的到来时，则会发生“接收就绪”（OP_ACCEPT）的事件
3.  一个SocketChannel通道有数据可读，则会发生“读就绪”（OP_READ）事件
4. 一个等待写入数据的SocketChannel通道，会发生写就绪（OP_WRITE）事件

#### SelectableChannel可选择通道

并不是所有的通道都可以被选择器监控或选择。比方说FileChannel文件通道就不能被选择器复用。

一个通道若能被选择，必需继承SelectableChannel类。

它提供了实现通道的可选择性所需要的公共方法。Java NIO中所有网络链接Socket套接字通道，都继承了SelectableChannel类，都是可选择的。而FileChannel文件通道，并没有继承SelectableChannel，因此不是可选择通道。

#### SelectionKey选择键

通道和选择器之间的监控关系，本质是一对多的关系。

​![image](assets/image-20230328113039-gevvtzk.png)​

Selectror并不直接去管理Channel，而是直接管理SelectionKey，通过SelectionKey与Channel发生关系。

Java NIO源码中规定了，一个Channel最多能向Selector注册一次，注册之后就形成了唯一的SelectionKey，然后被Selector管理起来。Selector有一个核心成员keys，专门用于管理注册上来的SelectionKey，Channel注册到Selector后所创建的那一个唯一的SelectionKey，添加在这个keys成员中，这是一个HashSet类型的集合。除了成员keys之外，Selector还有一个核心成员selectedKeys，用于存放已经发生了IO事件的SelectionKey。

除了弄清lectionKey和Channel、Selector之间的三角关系之后，还有一个核心问题，就是SelectionKey和IO事件之间的关系。实际上，SelectionKey是IO事件的记录者（或存储者），SelectionKey 有两个核心成员，存储着自己关联的Channel上的感兴趣IO事件和已经发生的IO事件

​![image](assets/image-20230328114141-cth1igr.png)​

Channel通道上可以发生多种IO事件，比如说读就绪事件、写就绪事件、新连接就绪事件，但是SelectionKey记录事件的成员却是一个整数类型。这样问题就来了，一个整数如何记录多个事件呢？答案是，通过比特位来完成的

​![image](assets/image-20230328114230-xs7stdh.png)​

通过SelectionKey的interestOps成员上相应的比特位，可以设置、查询关联的Channel所感兴趣的IO事件；通过SelectionKey的readyOps上相应的比特位，可以查询关联Channel所已经发生的IO事件

通道和选择器的监控关系注册成功后，Selector就可以查询就绪事件。具体的查询操作，是通过调用选择器Selector的select( )系列方法来完成。通过select系列方法，选择器会通过JNI，去进行底层操作系统的系统调用（比如select/epoll），可以不断地查询通道中所发生操作的就绪状态（或者IO事件），并且把这些发生了底层IO事件，转换成Java NIO中的IO事件，记录在的通道关联的SelectionKey的readyOps上。除此之外，发生了IO事件的选择键，还会记录在Selector内部selectedKeys集合中。

#### 选择器使用流程

1. 获取选择器实例
2. 将通道注册到选择器中
3. 轮询感兴趣的IO就绪事件（选择键集合）

##### 第一步：获取选择器实例

选择器实例是通过调用调用静态工厂方法open()来获取的，具体如下：

```java
//调用静态工厂方法 open()来获取 Selector 实例 
Selector selector = Selector.open();
```

Selector选择器的类方法open( )的内部，是向选择器SPI（SelectorProvider）发出请求，通过默认的SelectorProvider（选择器提供者）对象，获取一个新的选择器实例。

##### 第二步：将通道注册到选择器实例。

需要将通道注册到相应的选择器上，简单的示例代码如下：

```java
// 2.获取通道

ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

// 3.设置为非阻塞

serverSocketChannel.configureBlocking(false);

// 4.绑定连接

serverSocketChannel.bind(new InetSocketAddress(18899));

// 5.将通道注册到选择器上,并制定监听事件为：“接收连接”事件

serverSocketChannel.register(selector，SelectionKey.OP_ACCEPT);
```

上面通过调用通道的register方法，将 ServerSocketChannel通道注册到了一个选择器上

一个通道，并不一定要支持所有的四种IO事件。例如服务器监听通道ServerSocketChannel，仅仅支持Accept（接收到新连接）IO事件；而传输通道SocketChannel则不同，该类型通道不支持Accept类型的IO事件。

##### 第三步：选出感兴趣的IO就绪事件（选择键集合）

```java
//轮询，选择感兴趣的 IO 就绪事件（选择键集合） 
while (selector.select() > 0) 
{ 
Set selectedKeys = selector.selectedKeys(); 
Iterator keyIterator = selectedKeys.iterator(); 
while(keyIterator.hasNext()) 
{ 
SelectionKey key = keyIterator.next(); 
//根据具体的 IO 事件类型，执行对应的业务操作 
if(key.isAcceptable()) 
{ 
// IO 事件：ServerSocketChannel 服务器监听通道有新连接 }
else if (key.isConnectable()) { // IO 事件：传输通道连接成功 } 
else if (key.isReadable()) { // IO 事件：传输通道可读 } 
else if (key.isWritable()) { // IO 事件：传输通道可写 }
//处理完成后，移除选择键 keyIterator.remove(); 
}
}
```

处理完成后，需要将选择键从这个SelectionKey集合中移除，防止下一次循环的时候，被重复的处理

#### 实践案例


