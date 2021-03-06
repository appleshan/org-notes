# inode
linux下的inode是一种数据结构，他主要存储了以下元数据信息。
- 文件的字节数
- 文件拥有者的User ID
- 文件的Group ID
- 文件的读、写、执行权限
- 文件的时间戳，共有三个：ctime指inode上一次变动的时间，mtime指文件内容上一次变动的时间，atime指文件上一次打开的时间。
- 链接数，即有多少文件名指向这个inode
- 文件数据block的位置

inode在文件系统（fs）中以数组array的形式存在，占用特定大小的磁盘空间。每个inode大小也是固定的（毕竟是数组），常见大小是128/256byte。

最后一个属性block位置，其实是个复杂的存储结构，前12个块是直接存储，剩下的都是间接存储(毕竟固定大小，块太多了，没法固定inode的大小)。

![image](https://bolg.obs.cn-north-1.myhuaweicloud.com/1912/inode.png)

除了inode数组之外，还需要存一个映射表，文件名和inode标号（数组下标）的映射。
# 一次读取文件的过程
根据文件名从映射表找到inode标号，再从数组中找到inode，再根据权限信息看是否能读取，如果能，则根据block位置信息最终读取到文件的内容。
# block、sector
sector是扇区，是硬件上的最小存储单元，在磁盘生产的时候就决定了扇区的大小，早期常见的是512byte，比较新的硬盘都是4K大小了。

block是块，上面也提到过，文件在fs中是存放到块中的，块是fs对磁盘的一种抽象，块有标号，由1个或多个（2/4/8/16...）扇区组成。

块太大，会导致一个小文件也要占一整块这样的空间浪费，块太小又会使得inode数据结构变大，进而导致inode占用空间过大。所以是个权衡的值。
# inode用尽
inode是个数组，用`df -i`查看inode数量，用`ls -i`查看文件的inode下标。

如果巨量的小文件是会导致数组用完的，这就是早期docker的overlay文件系统的问题，后来换了overlay2解决了这个问题。

