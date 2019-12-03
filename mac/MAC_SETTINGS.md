# Mac常用设置


## 目录

* [修饰键](#修饰键)
* [显示个人用户文件夹](#显示个人用户文件夹)
* [修改用户名和用户文件夹](#修改用户名和用户文件夹)
* [修改HostName](#修改HostName)

## 修饰键



## 显示个人用户文件夹

访达->偏好设置->边栏->用户名打勾。

![访达](images/finder.png)


## 修改用户名和用户文件夹

* 打开终端,输入`sudo su`回车
* 输入登录密码(必须有密码才能执行)
* 输入`cd /Users`回车
* 输入`mv mac work`回车,mac就是你原来的个人目录名称(短名称),work是你想修改的名称。

  > 注：短名称必须全部小写、无空格且只包含字母。
  
  这时你会发现在访达里没有桌面、下载等等文件夹了,不要慌,因为原来的个人目录名称被修改了,电脑无法识别了,开始下一步。

* 打开**系统偏好设置**的**用户与群组(Users & Groups)**面板,来创建一个具有上一步骤中所使用的帐户名称或短名称的新用户。
  当系统提示**‘用户’文件夹中已经有名称为‘帐户名称’的文件夹。您想将该文件夹用作此用户帐户的个人文件夹吗？**时,点按**好**。
  
  > 注：这将更正个人文件夹中所有文件的所有权,并避免文件夹内容出现权限问题。

* 注销电脑。以新创建用户的身份登录(提示钥匙串问题点击更新钥匙串)。现在应该能够访问(桌面上、文稿中及该个人目录下其他文件夹中的)所有原始文件。
* 验证您的数据是否正常后,可通过**用户与群组**面板删除原始用户帐户。

  > 如果你发现有些文件夹无法访问提示无权限,右键文件夹->显示简介->共享与权限,
    点击右下角小锁(密码应该是无,直接回车就可以了),点+号添加你改名的管理员账号,接着权限改成读与写即可。


## 修改HostName

打开终端,执行下面命令,"tmp"是你想要修改的计算机名。

```
sudo scutil --set HostName tmp
```

> 执行过后命令,需要强行退出终端,重新打开就好了。