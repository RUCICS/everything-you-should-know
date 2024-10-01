# SSH

> 该教程假设你已经阅读了 shell 章节

ssh 是一种用于和服务器进行安全通信的协议，它的全称是 Secure Shell。ssh 有很多功能，其中最常用的功能是远程登录服务器。

如果你是 2024 年 ICS1 的学生，你可以访问 https://ics.ruc.panjd.net 来获取我们提供的服务器账号密码和服务器ip。

## 安装 ssh

目前大多数操作系统都会预装 ssh，如果你在终端中输入 `ssh -v` 报错，麻烦自行百度对应系统下 `ssh` 的安装方法。

## 使用 ssh

ssh 的基本使用方法是 `ssh <username>@<hostname>`，其中 `<username>` 是你在服务器上的用户名，`<hostname>` 是服务器的地址。

有时候服务器会修改默认端口，你可以使用 `-p` 参数来指定端口，比如 `ssh -p 2222 <username>@<hostname>`。

> 网络常识：
> 
> ip 是一台网络设备在互联网上的地址，我们可以通过 ip 连接到另一台网络设备（比如另一台电脑，手机）
>
> 域名是为了方便记忆而存在的，域名会指向某个 ip，比如输入 `ping baidu.com`，你应该会看到类似 `PING baidu.com (110.242.68.66) 56(84) bytes of data.` 的输出，说明 baidu.com 的服务器 ip 是 110.242.68.66。
>
> 一台设备只有一个 ip，为了区分你找这台设备上的哪个程序，我们使用端口号，端口号是一个 0-65535 的整数，比如 ssh 默认使用 22 端口，我们访问 http 网站的时候使用 80 端口，https 使用 443 端口。试着用 https://www.baidu.com:443 和 https://www.baidu.com:123 来访问百度，看看用错误的端口时会发生什么。
>
> 通常，还有一些特殊的 ip，比如 127.0.0.1 叫回环地址，指向的永远是自己。

总之，一般输入 `ssh <username>@<hostname>` 就可以登录到服务器了，然后会要求你输入密码，密码是不带回显的，即好像没反应，但其实是输入了，你只要保证你输入完后按下回车即可。

## 使用公钥登录

ssh 有两种登录方式，一种是使用密码登录，一种是使用公钥登录。通常公钥的安全性更高。

首先你需要在你登陆用的机器上生成一对公钥和私钥，比如要用你的笔记本电脑登陆服务器，则应该生成在笔记本电脑上。

直接输入 `ssh-keygen`

```shell
jarden@ROG-Flow-X13:~$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/jarden/.ssh/id_rsa):
```

记住它说的生成位置，在对应目录下，有两个文件 `id_rsa`，`id_rsa.pub`，前者就是私钥，后者就是公钥。有可能不叫这个名字，所以请参考实际情况。

注意中间叫你 `Enter passphrase (empty for no passphrase):`，这一步建议不输入，不然后面你每次用的时候都得输入一次这个密码，有点麻烦。

生成好后，复制 `id_rsa.pub` 的内容，然后登录到服务器，找到 `~/.ssh/authorized_keys` 文件，如果没有这个文件或没有这个文件夹，新建一个，然后把你复制的内容粘贴进去。

之后你应该就可以免密码登录了。这在 VSCode 中十分有用，不然你可能会频繁被请求输入密码。

## VSCode

掌握了终端的 ssh 连接，请查看这个小视频学习怎么在 VSCode 里连接服务器，并像操作本地文件一样操作服务器。

【vscode插件 remote ssh 远程开发方式 求你别再在服务器里直接改代码啦】 https://www.bilibili.com/video/BV13Z421T7oz/?share_source=copy_web&vd_source=7a01f17cc0a9cd546dd117967834433e