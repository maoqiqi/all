# Linux

> 作者：March    
> 链接：[Linux](https://github.com/maoqiqi/blog/blob/master/pages/os/linux.md)    
> 博客：http://blog.csdn.net/u011810138    
> 邮箱：fengqi.mao.march@gmail.com    
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。    


## 一、Linux 介绍

### Linux 的内核版本

Linux 的内核版本编号如下：

2.6.18-92.e15

主版本.次版本.释出版本-修改版本

* 主、次版本为奇数：开发中版本(development)，如：2.5.xx，适合开发特殊功能的环境。
* 主、次版本为偶数：稳定版本(stable)，如2.6.xx，适合于商业与家庭环境使用。

### Linux 的优缺点

优点

* 稳定的系统
* 免费或少许费用
* 安全性、漏洞的快速修补
* 多任务、多用户
* 用户与用户组的规划
* 相对比较不耗费资源的系统
* 适合需要小内核的嵌入式系统
* 整合度佳且多样的图形用户界面

缺点

* 没有特定的支持厂商
* 游戏的支持度不够
* 专业软件的支持度不够

### Linux 应用

企业环境

* 网络服务器
* 关键任务的应用（金融数据库、大型企业网管环境）
* 学术机构的高性能运算任务

个人环境

* 桌面计算机
* 手持系统（PAD、手机）
* 嵌入式系统

## 二、Linux 系统目录结构

* 根目录表示方式：`/`

* 根目录下常见目录

  * /bin：binary，二进制文件，可执行程序，shell 命令
  * /dev：device，在 Linux 下一切皆文件。（硬盘、显卡、显示器...）
  * /lib：Linux 运行的时候需要加载的一些动态库
  * /mnt：手动的挂载目录
  * /media：外设的自动挂载目录
  * /root：Linux 的超级用户的家目录
  * /usr：unix system resource
    * 头文件-stdio.h stdlib.h
    * 游戏
    * 用户安装的应用程序 /usr/local

  * /home：Linux 操作系统所有用户的家目录
    * 用户家目录：（宿主目录）/home/march

  * /etc：存放配置文件
    * /etc/passwd
    * /ect/group
    * man 5 passwd

  * /opt：安装第三方应用程序
  * /tmp：存放临时文件

## 三、shell 操作相关快捷键

* 命令或目录补齐
  * 快捷键：Tab

* 遍历历史记录
  * 显示命令历史列表：`history`
  * 显示上一条命令：`ctrl + p`
  * 显示下一条命令：`ctrl + n`
  * 执行命令历史列表的第num条命令：`!num`
  * 执行上一条命令：!!

* 光标移动
  * 光标向后移动一个字符：`ctrl + b`,相当与`<-`
  * 光标向前移动一个字符：`ctrl + f`,相当与`->`
  * 移动到当前单词的开头：`Esc + b`
  * 移动到当前单词的结尾：`Esc + f`
  * 移动到当前行的开头：`ctrl + a`
  * 移动到当前行的结尾：`ctrl + e`

* 字符删除
  * 删除光标前边的字符：`ctrl + h`,相当与`Backspace`
  * 删除光标后边的字符：`ctrl + d`,相当与`Delete`（光标覆盖的字符）
  * 删除光标前的字符串：`ctrl + u`（不包括自身）
  * 删除光标后的字符串：`ctrl + k`（包括自身）
  * 粘贴刚才所删除的字符:`ctrl + y`
  * 剪切光标所在处之前的一个词:`ctrl + w`（以空格、标点等为分隔符）

* 清屏
  * 快捷键：`ctrl + l`


## 四、文件目录相关命令

* `tree` 查看目录的内容
  * `tree` 查看当前目录
  * `tree dir` 查看指定目录
  * 需要安装：`sudo apt-get install tree`

* `ls` 查看文件或目录
  * -a 显示所有文件(隐藏文件：文件或目录前有一个点)
  * -l 显示文件详情

  **注意**

  -rwxrw-r-- 1 march march 1534696 3 26 12:01 redis-3.2.1.tar.gz

  * `-` ：文件的类型（7种）
    * 普通文件：-，如.txt、压缩包、可执行程序
    * 目录：d
    * 符号链接：l
    * 管道：p
    * 套接字：s
    * 字符设备：c，如键盘、鼠标
    * 块设备：b，如U盘、硬盘
  * rwxrw-r-- ：权限
    *  rwx ：文件夹所有者权限
    *  rw- ：文件所属组用户的权限
    *  r-- ：其他人对文件的操作权限
  * 1 ：硬链接计数
  * march ：文件所有者
  * march ：文件所属组
  * 1534696 ：文件的大小
    * 如果是目录：4k，目录本身的大小，不包括里面的内容
  * 3 26 12:01 ：日期
  * redis-3.2.1.tar.gz ：文件名

* `cd` 切换目录

* `pwd` 显示当前的工作目录

* `mkdir` 目录名 ：创建文件夹
  * -p ：创建多级目录

* `touch` 文件名：创建文件
  * 如果文件不存在，创建文件，空文件
  * 如果文件存在，更新文件的时间

7. `rmdir` 空目录名

  **注意**

## 五、相对路径和绝对路径

* 相对路径：从当前的目录开始表示

* 绝对路径：从根目录/开始表示的路径

* `.`和`..`
  * `.`  当前目录
  * `..` 当前目录的上一级目录

* 用户当前工作目录
  * march：当前登录的用户
  * @：at，在
  * linux：安装的时候指定的主机名
  * ~：用户的家目录
  * ~/demo：当前用户的工作目录
  * $：当前用户为普通用户
  * #：当前用户为超级用户


## 六、用户权限、用户和用户组


## 七、文件查找和检索


## 八、压缩包管理


## 九、软件的安装和卸载