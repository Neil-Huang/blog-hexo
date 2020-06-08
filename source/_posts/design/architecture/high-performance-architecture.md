---
title: 高性能架构
date: 2018-07-05 15:11
categories: ['设计', '架构']
tags: ['设计', '架构', '性能']
---

# 高性能架构

## 一、性能测试

> 性能测试是性能优化的前提和基础，也是性能优化结果的检查和度量标准。不同视角下的性能有不同标准，也有不同的优化手段。

### 性能指标

性能测试的主要指标有：

- **响应时间** - 响应时间(RT)是指从客户端发一个请求开始计时，到客户端接收到从服务器端返回的响应结果结束所经历的时间，响应时间由请求发送时间、网络传输时间和服务器处理时间三部分组成。
- **并发数** - 系统能同时处理的请求、事务数。
- **吞吐量** - **`QPS(每秒查询数)`**、**`TPS(每秒事务数)`**、**`HPS(每秒 HTTP 请求数)`**。
- **性能计数器** - 系统负载、对象与线程数、内存使用、CPU 使用、磁盘与网络 IO 等。这些指标也是系统监控的重要参数。

### 性能测试方法

- 性能测试
- 负载测试
- 压力测试
- 稳定性测试

### 性能测试报告

性能测试报告示例：

<div align="center">
<img src="https://upload-images.jianshu.io/upload_images/3101171-3d533a36f42608a1.png" />
</div>

### 性能优化策略

1.  **性能分析** - 如果请求响应慢，存在性能问题。需要对请求经历的各个环节逐一分析，排查可能出现性能瓶颈的地方，定位问题。检查监控数据，分析影响性能的主要因素：内存、磁盘、网络、CPU，可能是代码或架构设计不合理，又或者是系统资源确实不足。
2.  **性能优化** - 性能优化根据网站分层架构，大致可分为前端性能优化、应用服务性能优化、存储服务性能优化。

## 二。应用服务性能优化

### 分布式缓存

> 网站性能优化第一定律：优先考虑使用缓存优化性能。

#### 缓存原理

缓存指将数据存储在相对较高访问速度的存储介质中，以供系统处理。一方面缓存访问速度快，可以减少数据访问的时间，另一方面如果缓存的数据是经过计算处理得到的，那么被缓存的数据无需重复计算即可直接使用，因此缓存还起到减少计算时间的作用。

缓存的本质是一个内存 HASH 表。

缓存主要用来存放那些读写比很高、很少变化的数据，如商品的类目信息，热门词的搜索列表信息、热门商品信息等。

#### 合理使用缓存

缓存数据的选择：

- 不要存储频繁修改的数据
- 不要存储非热点数据
- 数据不一致与脏读：缓存有有效期，所以存在一定时间的数据不一致和脏读问题。如果不能接受，可以考虑使用数据更新立即更新缓存策略

需要考虑的缓存问题：

- **缓存雪崩** - 当缓存服务崩溃时，数据库会因为不能承受过大的压力而宕机。
- **缓存预热** - 对于一些明确的热点数据、元数据，如地名、类目信息等，可以在启动时加载到缓存进行预热。
- **缓存穿透** - 如果因不恰当的业务或恶意攻击持续高并发地请求某个不存在的数据，由于缓存中没有该数据，就会导致请求都落在数据库上，导致数据库最终崩溃。一个简单的策略是将不存在的数据也缓存起来（即缓存 null 值）。

### 异步操作

异步处理不仅可以减少系统服务间的耦合度，提高扩展性，事实上，它还可以提高系统的性能。异步处理可以有效减少响应等待时间，从而提高响应速度。

异步处理一般是通过分布式消息队列的方式。

异步处理可以解决以下问题：

- 异步处理
- 应用解耦
- 流量削锋
- 日志处理
- 消息通讯

### 使用集群

在高并发场景下，使用负载均衡技术为一个应用构建一个由多台服务器组成的服务器集群，将并发访问请求分发到多台服务器上处理，避免单一服务器因负载压力过大而响应缓慢，使用户请求具有更好的响应延迟特性。

### 代码优化

#### 多线程

从资源利用的角度看，使用多线程的原因主要有两个：IO 阻塞和多 CPU。

线程数并非越多越好，那么启动多少线程合适呢？

有个参考公式：

```
启动线程数 = (任务执行时间 / (任务执行时间 - IO 等待时间)) * CPU 内核数
```

最佳启动线程数和 CPU 内核数成正比，和 IO 阻塞时间成反比。

- 如果任务都是 CPU 计算型任务，那么线程数最多不要超过 CPU 内核数，因为启动再多线程，CPU 也来不及调度；
- 相反，如果是任务需要等待磁盘操作，网络响应，那么多启动线程有助于任务并罚赌，提高系统吞吐量。

##### 线程安全问题

线程安全问题时指多个线程并发访问某个资源，导致数据混乱。

解决手段有：

- **将对象设计为无状态对象** - 典型应用：Servlet 就是无状态对象，可以被服务器多线程并发调用处理用户请求。
- **使用局部对象**
- **并发访问资源时使用锁** - 但是引入锁会产生性能开销，应尽量使用轻量级的锁。

#### 资源复用

应该尽量减少那些开销很大的系统资源的创建和销毁，如数据库连接、网络通信连接、线程、复杂对象等。从编程角度，资源复用主要有两种模式：单例模式和对象池。

#### 数据结构

根据具体场景，选择合适的数据结构。

#### 垃圾回收

如果 Web 应用运行在 JVM 等具有垃圾回收功能的环境中，那么垃圾回收可能会对系统的性能特性产生巨大影响。立即垃圾回收机制有助于程序优化和参数调优，以及编写内存安全的代码。

## 三、存储性能优化

### 数据库读写分离

**读写分离的基本原理是将数据库读写操作分散到不同的节点上**

读写分离的基本实现是：

- 数据库服务器搭建主从集群，一主一从、一主多从都可以。
- 数据库主机负责读写操作，从机只负责读操作。
- 数据库主机通过复制将数据同步到从机，每台数据库服务器都存储了全量数据。
- 业务服务器将写操作发给数据库主机，将读操作发给数据库从机。
- 主机会记录请求的二进制日志，然后推送给从库，从库解析并执行日志中的请求，完成主从复制。这意味着：复制过程存在时延，这段时间内，主从数据可能不一致。

读写分离存在两个问题：**数据一致性**和**分发机制**。

解决方法：

#### 数据一致性

（1）写操作后的读操作指定发给数据库主服务器。缺点：业务捆绑痕迹太深，容易出错。

（2）读从机失败后再读一次主机。缺点：如果存在很多二次查询，大大增加了主机压力。

（3）关键业务读写操作全部指向主机，非关键业务采用读写分离。

#### 分发机制

数据库读写分离后，一个 SQL 请求具体分发到哪个数据库节点？一般有两种分发方式：客户端分发和中间件代理分发。

客户端分发，是基于程序代码，自行控制数据分发到哪个数据库节点。更细一点来说，一般程序中建立多个数据库的连接，根据一定的算法，选择合适的连接去发起 SQL 请求。这种方式也被称为客户端中间件，代表有：jdbc-sharding。

中间件代理分发，指的是独立一套系统出来，实现读写操作分离和数据库服务器连接的管理。中间件对业务服务器提供 SQL 兼容的协议，业务服务器无须自己进行读写分离。对于业务服务器来说，访问中间件和访问数据库没有区别，事实上在业务服务器看来，中间件就是一个数据库服务器。代表有：Mycat。

### 数据库分库分表

### 机械键盘和固态硬盘

考虑使用固态硬盘替代机械键盘，因为它的读写速度更快。

### B+数和 LSM 树

传统关系数据库的数据库索引一般都使用两级索引的 **B+ 树** 结构，树的层次最多三层。因此可能需要 5 次磁盘访问才能更新一条记录（三次磁盘访问获得数据索引及行 ID，然后再进行一次数据文件读操作及一次数据文件写操作）。

由于磁盘访问是随机的，传统机械键盘在数据随机访问时性能较差，每次数据访问都需要多次访问磁盘影响数据访问性能。

许多 Nosql 数据库中的索引采用 **LSM 树** 作为主要数据结构。LSM 树可视为一个 N 阶合并树。数据写操作都在内存中进行。在 **LSM 树上进行一次数据更新不需要磁盘访问，速度远快于 B+ 树**。

### RAID 和 HDFS

RAID 是 Redundant Array of Independent Disks 的缩写，中文简称为独立冗余磁盘阵列。

RAID 是一种把多块独立的硬盘（物理硬盘）按不同的方式组合起来形成一个硬盘组（逻辑硬盘），从而提供比单个硬盘更高的存储性能和提供数据备份技术。

HDFS(分布式文件系统) 更被大型网站所青睐。它可以配合 `MapReduce` 并发计算任务框架进行大数据处理，可以在整个集群上并发访问所有磁盘，无需 RAID 支持。

HDFS 对数据存储空间的管理以数据块（Block）为单位，默认为 64 MB。所以，HDFS 更适合存储较大的文件。

## 前端性能优化

### 浏览器访问优化

1.  **减少 HTTP 请求** - HTTP 请求需要建立通信链路，进行数据传输，开销高昂，所以减少 HTTP 请求数可以有效提高访问性能。减少 HTTP 的主要手段是合并 Css、JavaScript、图片。
2.  **使用浏览器缓存** - 因为静态资源文件更新频率低，可以缓存浏览器中以提高性能。设置 HTTP 头中的 `Cache-Control` 和 `Expires` 属性，可设定浏览器缓存。
3.  **启用压缩** - 在服务器端压缩静态资源文件，在浏览器端解压缩，可以有效减少传输的数据量。由于文本文件压缩率可达 80% 以上，所以可以对静态资源，如 Html、Css、JavaScrip 进行压缩。
4.  **CSS 放在页面最上面，JavaScript 放在页面最下面** - 浏览器会在下载完全部的 Css 后才对整个页面进行渲染，所以最好的做法是将 Css 放在页面最上面，让浏览器尽快下载 Css；JavaScript 则相反，浏览器加载 JavaScript 后立即执行，可能会阻塞整个页面，造成页面显示缓慢，因此 JavaScript 最好放在页面最下面。
5.  **减少 Cookie 传输** - Cookie 包含在 HTTP 每次的请求和响应中，太大的 Cookie 会严重影响数据传输。

### CDN

CDN 一般缓存的是静态资源。

CDN 的本质仍然是一个缓存，而且将数据缓存在离用户最近的地方，使用户已最快速度获取数据，即所谓网络访问第一跳。

<div align="center">
<img src="http://dunwu.test.upcdn.net/cs/java/javaweb/architecture/cdn.png!zp" width="640" />
</div>


### 反向代理

传统代理服务器位于浏览器一侧，代理浏览器将 HTTP 请求发送到互联网上，而反向代理服务器位于网站机房一侧，代理网站服务器接收 HTTP 请求。

<div align="center">
<img src="http://dunwu.test.upcdn.net/cs/java/javaweb/architecture/reverse-proxy.jpg!zp" width="640" />
</div>


反向代理服务器可以配置缓存功能加速 Web 请求，当用户第一次访问静态内容时，静态内容就会被缓存在反向代理服务器上。

反向代理还可以实现负载均衡，通过负载均衡构建的集群可以提高系统总体处理能力。

因为所有请求都必须先经过反向代理服务器，所以可以屏蔽一些攻击 IP，达到保护网站安全的作用。

## 参考资料

- [《大型网站技术架构：核心原理与案例分析》](https://item.jd.com/11322972.html)