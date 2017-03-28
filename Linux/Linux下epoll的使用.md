# Linux下epoll的使用

## epoll的工作方式
epoll有两种工作方式：
* LT:level triggered,即满足某种状态则触发，类似于数字电路中的高／低电平触发
* ET:edge triggered ,即状态发生变化时触发，类似于数字电路的上升／下降沿触发

epoll默认的工作模式是LT。使用LT时，如果事件发生后应用程序未处理，epoll会继续报告，直到事件被处理；而
使用ET模式时，epoll只会通知一次而不会重复触发，因此应用程序必须处理完。由于ET模式不会重复触发，所以效率较高。需要注意的是，使用ET模式读取数据时，一定要读到无数据为止，否则epoll不会再次触发。
LT支持阻塞IO和非阻塞IO,ET只支持非阻塞IO。

## epoll系统调用
epoll实际上是一系列系统调用，Linux提供了Ｃ函数的封装：

`int epoll_create(int size);`

创建epoll：
size:监听的对象数量
返回值：epoll的文件描述符

`int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);`

epoll控制：
ｅpfd:epoll文件描述符
op:操作类型：
* EPOLL_CTL_ADD：注册新的fd到epfd中
* EPOLL_CTL_MOD：修改已经注册的fd的监听事件
* EPOLL_CTL_DEL：从epfd中删除一个fd
fd：被操作对象的文件描述符
event:需要监听的事件

`int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);`

阻塞当前线程，直到有事件发生
epfd:epoll文件描述符
events:需要处理的事件数组头指针
maxevents:events数组长度，即最多有多少个事件要处理
timeout:超时时间，以毫秒计。０立即返回，－１永不超时
返回值：就绪的事件数量，返回０表示已超时
