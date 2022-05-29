---
title: "创建空文件显示No space left on device"
date:   2022-05-25  16:57:00 +0800
layout: post
tag:
- Linux
categories:
- Os
---

创建空文件显示No space left on device

------

## 背景
同事写的代码逻辑有问题，程序运行一夜以后，磁盘占用率100%，随便统计了一下创建的垃圾文件100W+，忍住怒火，给他收拾烂摊子。

## 准备
由于文件太多，使用rm删除速度缓慢，参考一篇腾讯云的一篇文章：《Linux 下删除大量文件效率对比，看谁删的快！》,以删除50W空文件来测试，使用rsync，采用替换原理删除文件较为方便和高效，部分博文写的有问题，被坑到了（自己太菜了）。
> rm：文件数量太多，不可用
> find with -exec 50万文件耗时43分钟
> find with -delete 9分钟
> Perl  16s
> Python 9分钟
> rsync with -delete  16s

删除命令：
```
rsync --delete-before -d -a -H -v --progress --stats plugins/ bin/
```
* bin目录是需要删除的，plugins目录是空目录
* --delete-before 表示在传输过程中删除
* --stats 显示文件传输状态
* -v 详细输出
* -d 不重复传输目录
* -a 归档模式，删除时没啥用
* -H 保持硬链接文件
* --progress 传输时显示传输过程
由于文件比较多，删除耗费时间过长，设备又等着释放空间启动其他应用程序，如果使用rsync进行删除，最后一步需要移除plugins文件，否则inode不会释放，查看磁盘，占用率仍然为100%。
rsync能快熟进行删除，采用替换的原理进行，执行完rsync命令后，需要移除空文件，才能释放文件inode，否则就会出现，文件已删除，但是会显示“No space left on device”，现象如下：
![No space left on device](/img/20220525-01.png)    
rsync删除过程，文件占用率未发生变化：
![磁盘文件占用率未发生变化](/img/20220525-02.png)
* 此处有个知识点，由于磁盘使用btrfs文件系统，BTRFS并不以其他文件系统的方式使用Inodes，也没有预先确定的限制，因此无需计算。 由于BTRFS不使用Inodes，BTRF的Inode计数始终显示为零。
删除同步文件目录plugins后，磁盘占用率立刻恢复正常：
![磁盘文件占用率恢复正常](/img/20220525-03.png)

## 总结

*   rsync删除速度迅速，但是需要移除同步文件夹才会释放inode，否则磁盘无法写入文件
*   btrfs使用不当，理论上建立快照，通过恢复快照即可，有些暴殄天物

## 参考
- [https://cloud.tencent.com/developer/article/1647290](https://cloud.tencent.com/developer/article/1647290)【Linux 下删除大量文件效率对比，看谁删的快！】
- [https://www.thegeekdiary.com/command-df-i-shows-inode0-on-btrfs-file-system/](https://www.thegeekdiary.com/command-df-i-shows-inode0-on-btrfs-file-system/)【Command ‘df -i’ Shows ‘Inode=0’ on BTRFS File System】
- [https://wiki.archlinux.org/title/Btrfs](https://wiki.archlinux.org/title/Btrfs)【Btrfs】