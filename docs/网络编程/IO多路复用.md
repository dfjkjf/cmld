---
sort: 5
---

# IO多路复用

当进程处理多个网络连接时一般使用多线程处理每个连接

由于多线程切换开销较大，所以利用**单线程**处理**多个网络连接**

基本操作是**轮询**
```c
// 循环遍历文件描述符集合
while (true)
{
   for (int fd : fds)
   {
      if (fd can read)
      {
         read(fd, ...);
      }
   }
}
```
这样单线程也能处理多个网络连接，但是有很大缺点：
1. 判断每个文件描述符是否可读会产生用户态内核态切换开销
2. 线程轮询霸占CPU造成资源浪费

## 利用内核判断fd是否可读：select

在IO多路复用中，`select`函数是用来监视多个文件描述符的读写等事件是否就绪的函数。当调用`select`函数时，如果任何一个文件描述符就绪（即有数据可读、可写或者有异常等），`select`函数会返回。

在`select`函数执行期间，如果没有任何文件描述符就绪，线程会阻塞。


```c
// 循环遍历文件描述符集合
while (true)
{
   FD_ZERO(&rset);
   for (int fd : fds)
      FD_SET(fd, &rset);

   // max是文件描述符中最大的
   // select会将文件描述符集合复制到内核中
   select(max+1, &rset, NULL, NULL, NULL);
   
   for (int fd : fds)
   {
      if (FD_ISSET(fd, &rset))
      {
         read(fd, ...);
      }
   }
}
```

但是`select`同样有缺点：
1. 能够检测的最大文件描述符个数是1024。（因为select使用固定大小的位图来表示文件描述符集合，每个位图通常包含1024位。）
2. fd_set不可重用
3. 依旧有用户态内核态切换开销
4. 阻塞解除后还要遍历才能确定哪个fd可读

## linux平台函数poll

基于结构体存储fd，解决了select的1,2两点缺点

```c
#include <poll.h>
// 每个委托poll检测的fd都对应这样一个结构体
struct pollfd {
    int   fd;         /* 委托内核检测的文件描述符 */
    short events;     /* 委托内核检测文件描述符的什么事件 */
    short revents;    /* 文件描述符实际发生的事件 -> 传出 */
};

for (auto fd : plfds)
{
   fd.fd = $(fd);
   fd.events = POLLIN;
   fd.revents = 0;
}

while (true)
{
   poll(plfds, plfds_size, timeout);

   for (auto fd : plfds)
   {
      if (fd.revents & POLLIN)
      {
         fd.revents = 0;
         read(fd, ...);
      }
   }
}
```

## 红黑树管理fds：epoll

解决3，4问题
- epoll中内核和用户区使用的是共享内存（基于mmap内存映射区实现），省去了不必要的内存拷贝。
- epoll将已就绪的文件描述符重排到集合前面，并返回个数，无需再次检测

```c
// 联合体, 多个变量共用同一块内存        
typedef union epoll_data {
 	void        *ptr;
	int          fd;	// 通常情况下使用这个成员, 和epoll_ctl的第三个参数相同即可
	uint32_t     u32;
	uint64_t     u64;
} epoll_data_t;

struct epoll_event {
	uint32_t     events;      /* Epoll events */
	epoll_data_t data;        /* User data variable */
};

while (true)
{
   readyfd_size = epoll(epfd, events, events_size, timeout);

   for (int i = 0; i < readyfd_size; i++)
      read(events[i].data.fd, ...);
}
```

### `epoll`的工作模式

`epoll`是Linux中用于IO多路复用的一个高效事件通知机制。`epoll`有两种工作模式：水平触发（Level Triggered，LT）和边缘触发（Edge Triggered，ET）。

以下是水平触发和边缘触发的主要区别：
#### 水平触发（LT）
- **触发条件**：当文件描述符对应的读/写缓冲区不为空/不满时，如果进行`epoll_wait`调用，会立即返回就绪的文件描述符。
- **行为**：如果用户没有完全处理缓冲区中的数据，则`epoll`会再次通知。
- **适用性**：适用于大多数情况，特别是当你不确定能否一次性处理完所有数据时。
- **处理方式**：可以多次读取或者写入，直到缓冲区空或者满。

#### 边缘触发（ET）
- **触发条件**：只有当文件描述符的状态发生变化时，例如从不可读变为可读，或者从不可写变为可写，`epoll_wait`才会返回。
- **行为**：`epoll`只会在文件描述符状态变化时通知一次，即使缓冲区中还有数据剩余，也不会再次通知。
- **适用性**：适用于需要高效率处理大量事件，且能够确保一次处理完缓冲区中所有数据的情况。
- **处理方式**：需要尽可能多地读取或者写入，直到`EAGAIN`错误出现，表示没有更多数据可以处理。

#### 总结
- **LT模式**更简单，更安全，因为它允许程序员错过事件后再次得到通知。但是，这可能会引入不必要的唤醒和检查，降低效率。
- **ET模式**效率更高，因为它减少了`epoll`事件的触发次数，但是需要更仔细地处理每个事件，确保不会因为错过事件而导致数据丢失。

选择哪一种模式，需要根据具体的应用场景和需求来定。通常，ET模式需要更复杂的处理逻辑来确保不会漏掉任何事件，因此在某些情况下可能更难于正确实现。

因为套接字操作默认是阻塞的，当读缓冲区数据被读完之后，读操作就阻塞了也就是调用的read()/recv()函数被阻塞了，当前线程被阻塞之后就无法处理其他文件描述符了。

要解决阻塞问题，就需要将套接字默认的阻塞行为修改为非阻塞，

### ET必须为非阻塞

在ET（边缘触发）模式下，为了避免数据丢失，可以遵循以下策略：
1. **循环读取或写入直到EAGAIN**：
   - 当读取数据时，应该在一个循环中读取，直到`read`或类似函数返回`EAGAIN`错误，这表示当前没有更多数据可以读取。
   - 当写入数据时，同样应该在循环中写入，直到`write`或类似函数返回`EAGAIN`错误，表示缓冲区已满。
2. **处理非阻塞套接字**：
   - 确保`epoll`管理的套接字是非阻塞的。这样，在读取或写入时，如果缓冲区没有数据或者满了，操作会立即返回`EAGAIN`，而不是阻塞。
3. **处理错误情况**：
   - 在读取或写入时，检查返回值是否表示错误（例如`ECONNRESET`、`EPIPE`等），并相应地处理这些情况。

```c
while (1) {
    int n, i;
    struct epoll_event events[MaxEvents];
    // 等待事件发生
    n = epoll_wait(epfd, events, MaxEvents, -1);
    for (i = 0; i < n; i++) {
        if (events[i].events & EPOLLIN) { // 如果是可读事件
            int fd = events[i].data.fd;
            char buffer[BUF_SIZE];
            while (1) {
                // 循环读取直到没有更多数据
                ssize_t count = read(fd, buffer, BUF_SIZE);
                if (count == -1) {
                    if (errno != EAGAIN) {
                        perror("read error");
                        close(fd); // 处理错误
                    }
                    break; // 没有更多数据可以读取
                } else if (count == 0) {
                    // EOF，对方关闭连接
                    close(fd);
                    break;
                }
                // 处理读取到的数据
                handle_data(buffer, count);
            }
        }
        // 处理其他事件...
    }
}
```

在上面的代码中，`handle_data`函数负责处理从套接字读取到的数据。循环中的`read`调用会一直执行，直到没有更多数据可以读取（`EAGAIN`错误）。如果读取到0字节，意味着对方关闭了连接，因此我们也关闭套接字并退出循环。
遵循这些策略，可以在ET模式下有效地避免数据丢失。
