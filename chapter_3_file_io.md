## 引言
本章介绍不带缓冲的I/O。不带缓冲I/O指的是每个`read`和`write`都调用内核中的一个系统调用。  
只要涉及在多个进程间共享资源，原子操作的概念就变得非常重要，本章会讨论多个进程如何共享文件。

## 文件描述符
- 打开的文件都是通过文件描述符
- 非负整数
- 打开/创建一个文件时，内核向进程返回一个文件描述符
- `shell`把文件描述符0与进程的标准输入关联，1与标准输出关联，2与标准错误关联

## `open` 和 `openat` 函数
```c
#include <fcntl.h>

int open(const char *path, int oflag, ... /* mode_t mode */ );
int openat(int fd, const char *path, int oflag, ... /* mode_t mode */ );

/* Both return: file descriptor if OK, −1 on error */
```
oflag参数可用来说明此函数的多个选项，一个或多个常量进行“或”运算构成oflag参数  
必须项：  
- `O_RDONLY`
- `O_WRONLY`
- `O_RDWR`
- `O_EXEC`
- `O_SEARCH`

## `create` 函数 
```c
#include <fcntl.h>

int creat(const char *path, mode_t mode);

/* Returns: file descriptor opened for write-only if OK, −1 on error */
```
等效于:
```c
open(path, O_WRONLY | O_CREAT | O_TRUNC, mode);
```
创建一个可读写的:
```c
open(path, O_RDWR | O_CREAT | O_TRUNC, mode);
```

## `close` 函数
```c
#include <unistd.h>

int close(int fd);

/* Returns: 0 if OK, −1 on error */
```
关闭一个文件时还会释放该进程加在该文件上的所有记录锁。  
当一个进程终止时，内核自动关闭它所有的打开文件。  

## `lseek` 函数
每个打开文件都有一个对应关联的"当前文件偏移量"，它通常是非负整数，通常读写操作都是从文件当前偏移量处开始，并使偏移量增加所读写的字节数，  
当打开一个文件时，除非指定append，否则偏移量是0
```c
#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);

/* Returns: new file offset if OK, −1 on error */
```
- SEEK_SET: 偏移量是距离文件开始offset个字节
- SEEK_CUR: 当前值加offset，offset可以为负
- SEEK_END: 文件长度加offset，offset可以为负

lseek仅将当前偏移量记录内核中，不会引起任何I/O操作。  
文件偏移量大于文件当前长度，对下一次写将加长该文件，并在文件中构成空洞，没有写过的字节被读为0。  
可以用`od -c`命令打印文件内容。  

