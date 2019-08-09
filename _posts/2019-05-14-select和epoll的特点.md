---
layout:     post                    # 使用的布局（不需要改）
title:      select和epoll的特点              # 标题 
subtitle:   io模型 #副标题
date:       2019-05-14              # 时间
author:     BY                      # 作者
header-img: img/wkj.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 生活
---

## Hey
##select:


1.select能监听的文件描述符个数受限于FD_SETSIZE,一般为1024，单纯改变进程打开的文件描述符个数并不能改变select监听文件个数


2.解决1024以下客户端时使用select是很合适的，但如果链接客户端过多，select采用的是轮询模型，会大大降低服务器响应效率，不应在select上投入更多精力


#include <sys/select.h>

/* According to earlier standards */

#include <sys/time.h>

#include <sys/types.h>

#include <unistd.h>

int select(int nfds, fd_set *readfds, fd_set *writefds,

fd_set *exceptfds, struct timeval *timeout);

nfds: 监控的文件描述符集里最大文件描述符加1，因为此参数会告诉内核检测前多少个文件描述符的状态

readfds：监控有读数据到达文件描述符集合，传入传出参数

writefds：监控写数据到达文件描述符集合，传入传出参数

exceptfds：监控异常发生达文件描述符集合,如带外数据到达异常，传入传出参数

timeout：定时阻塞监控时间，3种情况:


1.NULL，永远等下去


2.设置timeval，等待固定时间


3.设置timeval里时间均为0，检查描述字后立即返回，轮询.


struct timeval {

long tv_sec; /* seconds */

long tv_usec; /* microseconds */

};

void FD_CLR(int fd, fd_set *set); 把文件描述符集合里fd清0

int FD_ISSET(int fd, fd_set *set); 测试文件描述符集合里fd是否置1

void FD_SET(int fd, fd_set *set); 把文件描述符集合里fd位置1

void FD_ZERO(fd_set *set); 把文件描述符集合里所有位清0

pselect 给出pselect原型，此模型用的不多，有需要的同学可参考select模型自行编写C/S

#include <sys/select.h>

int pselect(int nfds, fd_set *readfds, fd_set *writefds,

fd_set *exceptfds, const struct timespec *timeout,

const sigset_t *sigmask);

struct timespec {

long tv_sec; /* seconds */

long tv_nsec; /* nanoseconds */

};


用sigmask替代当前进程的阻塞信号集，调用返回后还原原有阻塞信号集

##poll

include <poll.h>

int poll(struct pollfd *fds, nfds_t nfds, int timeout);

struct pollfd {

int fd; /* 文件描述符*/

short events; /* 监控的事件*/

short revents; /* 监控事件中满足条件返回的事件*/

};

POLLIN普通或带外优先数据可读,即POLLRDNORM | POLLRDBAND

POLLRDNORM-数据可读

POLLRDBAND-优先级带数据可读

POLLPRI 高优先级可读数据

POLLOUT普通或带外数据可写

POLLWRNORM-数据可写

POLLWRBAND-优先级带数据可写

POLLERR 发生错误

POLLHUP 发生挂起

POLLNVAL 描述字不是一个打开的文件

nfds 监控数组中有多少文件描述符需要被监控

timeout 毫秒级等待

-1：阻塞等，#define INFTIM -1 Linux中没有定义此宏

0：立即返回，不阻塞进程

>0：等待指定毫秒数，如当前系统时间精度不够毫秒，向上取值

如果不再监控某个文件描述符时，可以把pollfd中，fd设置为-1，poll不再监控此

pollfd，下次返回时，把revents设置为0。

ppoll GNU定义了ppoll(非POSIX标准)，可以支持设置信号屏蔽字，大家可参考poll模型自行实现C/S


##epoll

epoll是Linux下多路复用IO接口select/poll的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率，因为它会复用文件描述符集合来传递结果而不用迫使开发者每次等待事件之前都必须重新准备要被侦听的文件描述符集合，另一点原因就是获取事件的时候，它无须遍历整个被侦听的描述符集，只要遍历那些被内核IO事件异步唤醒而加入Ready队列的描述符集合就行了。目前epell是linux大规模并发网络程序中的热门首选模型。

epoll除了提供select/ poll那种IO事件的电平触发（Level Triggered）外，还提供了边沿触发（Edge Triggered），这就使得用户空间程序有可能缓存IO状态，减少epoll_wait/epoll_pwait的调用，提高应用程序效率。

一个进程打开大数目的socket描述符

cat /proc/sys/fs/file-max

设置最大打开文件描述符限制

最大打开文件个数设置

sudo vi /etc/security/limits.conf

写入以下配置,soft软限制，hard硬限制

* soft nofile 65536

* hard nofile 100000


epoll API


1.创建一个epoll句柄，参数size用来告诉内核监听的文件描述符个数，跟内存大小有关

int epoll_create(int size)

size：告诉内核监听的数目


2.控制某个epoll监控的文件描述符上的事件：注册、修改、删除。


#include <sys/epoll.h>

int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)

epfd：为epoll_creat的句柄

op：表示动作，用3个宏来表示：

EPOLL_CTL_ADD(注册新的fd到epfd)，

EPOLL_CTL_MOD(修改已经注册的fd的监听事件)，

EPOLL_CTL_DEL(从epfd删除一个fd)；

event：告诉内核需要监听的事件

struct epoll_event {

__uint32_t events; /* Epoll events */

epoll_data_t data; /* User data variable */

};

EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）

EPOLLOUT：表示对应的文件描述符可以写

EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）

EPOLLERR：表示对应的文件描述符发生错误

EPOLLHUP：表示对应的文件描述符被挂断；

EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的

EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里

3.等待所监控文件描述符上有事件的产生，类似于select()调用。

#include <sys/epoll.h>

int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout)

events：用来从内核得到事件的集合，

maxevents：告之内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size，

timeout：是超时时间

-1：阻塞

0：立即返回，非阻塞

>0：指定微秒

返回值：成功返回有多少文件描述符就绪，时间到时返回0，出错返回-1

