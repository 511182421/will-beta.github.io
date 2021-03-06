---
title: 性能测试工具包sysstat
categories: [测试]
tags: [测试]
sitemap:
    lastmod: 2015-10-9T12:49:30-05:00
---



概述
========================================================================================

* 包括多个工具
* 安装：yum install sysstat fio






守护活动
========================================================================================

cat /etc/cron.d/sysstat

```
# 每隔10分钟运行一次系统活动统计工具，存储至/var/log/sa/sa*文件中
*/10 * * * * root /usr/lib64/sa/sa1 1 1
# 0 * * * * root /usr/lib64/sa/sa1 600 6 &
# 每天的23:53生成一个进程活动日统计数据，存储至/var/log/sa/sar*文件中
53 23 * * * root /usr/lib64/sa/sa2 -A
```






统计CPU情况
========================================================================================

sar -p -f /var/log/sa/sa25（查看各CPU负载的统计情况）
-----------------------------------------------------------

### 概念

* PRI：进程优先权，代表这个进程可被执行的优先级；其值越小优先级就越高，越早被执行
* NI：进程NICE值，进程可被执行的优先级的修正数值；PRI(new)=PRI(old)+nice


### 数据

* %nice：改过优先级的进程CPU占用率
* %steal：管理程序（hypervisor）为另一个虚拟进程提供服务而等待虚拟CPU的百分比，值越高表示CPU负载越高
* %iowait：等待IO而产生的CPU空闲率，值越高表示IO负载越高



sar -q -f /var/log/sa/sa25（查看任务队列的统计情况）
-----------------------------------------------------------

### 数据

* runq-sz：运行中的任务总数
* plist-sz：队列中的任务总数
* lavg-1,lavg-5,lavg-15：前1分钟、前5分钟、前15分钟的系统负载；通过计算执行中running的任务+可执行runnable（等待执行）的任务的个数的和的平均值得到的；当值>CPU总数时表示CPU压力大。







统计内存情况
========================================================================================

sar -r -f /var/log/sa/sa25（查看内存的统计情况）
-----------------------------------------------------------

### 概念

* buffer/cache是为了提高文件读取性能的缓存


### 数据

* page cache：实际上是针对文件系统的，是文件的缓存
* buffer cache：是针对磁盘块的缓存
* kbcommit：为了保证程序的正常运行需要的内存百分比



sar -B -f /var/log/sa/sa25（查看内存页的统计情况）
-----------------------------------------------------------

### 概念

* fault：内存被有命中就是一个fault
* majflt=major faults：当发生major faults的时候就会发生内存换页


### 数据

* pagin/s：每秒从磁盘或SWAP置换到内存的大小
* pagout/s：每秒从内存置换到磁盘或SWAP的大小
* fault/s：每秒钟系统产生的缺页数，即主缺页(major)和副缺页(minor)之和。主缺页是需要将数据换入内存，而副缺页不需要。
* majflt/s：每秒钟产生的主缺页数
* pgfree/s：每秒钟被放入空闲队列中的页个数
* pgscank/s：每秒钟被kswapd扫描的页个数
* pgscand/s：每秒钟直接被扫描的页个数
* pgsteal/s：每秒钟从cache中被清除来满足内存需要的页个数
* %vmeff：每秒钟清除的页（pgsteal）占总扫描页（pgscank+pgscand）



sar -B -f /var/log/sa/sa25（查看换页的统计情况）
-----------------------------------------------------------

### 数据

* pswpin/s
* pswpout/s







统计IO情况
========================================================================================

sar -b -f /var/log/sa/sa25（查看IO的统计情况）
-----------------------------------------------------------

### 数据

* tps：每秒钟物理设备的IO请求次数
* rtps：每秒钟从物理设备读出的请求次数
* rtps：每秒钟从物理设备写入的请求次数
* bread/s：每秒钟从物理设备读出的数据量，单位为块/秒
* bwrtn/s：每秒钟从物理设备写入的数据量，单位为块/秒



sar -d -f /var/log/sa/sa25（查看各块设备的运行情况）
-----------------------------------------------------------

### 数据

* rd_sec/s：每秒读扇区的次数
* avgrq-sz：平均每次设备IO操作的数据大小
* avgqu-sz：磁盘请求队列的平均长度
* %util：IO请求占CPU的百分比，比率越大，说明越饱和








统计网络情况
========================================================================================

sar -n DEV -f /var/log/sa/sa25（查看网络的统计情况）
-----------------------------------------------------------

### 数据

* IFACE：网卡接口名
* rxpck/s：每秒钟接收的数据包
* rxbyt/s：每秒钟接收的字节数
* rxcmp/s：每秒钟接收的压缩数据包
* rxmcst/s：每秒钟接收的多播数据包
* rxerr/s：每秒钟接收的坏数据包
* coll/s：每秒钟的冲突数
* rxdrop/s：因为缓冲充满，每秒钟丢弃的已接收数据包数







磁盘读写性能的测试工具fio
========================================================================================

安装
-----------------------------------------------------------

yum install fio



使用
-----------------------------------------------------------

### 命令

fio -filename=/data/test -direct=1 -iodepth 1 -thread -rw=randrw -ioengine=psync -bs=16k -size 2G -numjobs=10 -runtime=30 -group_reporting -name=mytest13


### 数据

* iops

