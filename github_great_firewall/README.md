<!-- vscode-markdown-toc -->
* 1. [为什么 G 网站无法访问](#G)
* 2. [一些基础知识](#)
	* 2.1. [什么是代理](#-1)
	* 2.2. [什么是端口（port）](#port)
	* 2.3. [什么是 IP](#IP)
	* 2.4. [代理的种类](#kind)
	* 2.5. [怎么使用代理](#how)
	* 2.6. [什么是系统代理](#what)
* 3. [WSL 怎么配置为镜像模式](#WSL)
* 4. [从 https 协议 clone/pull/fetch G 网站的代码](#httpsclonepullfetchG)
* 5. [从 ssh 协议 clone/pull/fetch G 网站的代码](#sshclonepullfetchG)
	* 5.1. [不配代理的神奇方案（也许未来这个方法会失效）](#magic)
	* 5.2. [配代理访问的方法](#proxy)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

> 如果有任何不懂的地方，请连着不理解地方的上下文一起丢给 AI，它总能解释清楚



###  1. <a name='G'></a>为什么 G 网站无法访问

这个网站受神秘电波影响，访问时好时坏，看神秘电波的心情，会出现你之前能访问，你啥都没做又不可以了的情况

神秘电波的干扰方式很多样，有看你访问某个网站就全拦住的，也有那种检查一下你是什么协议，如果你是访问网页，那就阻止，如果你是 push clone 这种，就放你过去

后者比较耗资源，如果干这活的服务器压力太大，可能就会搞成一刀切

###  2. <a name=''></a>一些基础知识

####  2.1. <a name='-1'></a>什么是代理

代理指将你的网络请求，经过一个中间服务器再发送到目标服务器，这个中间服务器就是代理服务器

如今我们很少直接访问代理服务器，而是你本地会有一个代理软件，这个软件充当了一级代理服务器，它连接到真正的代理服务器，然后你的网络请求就会经过这个软件再发送到真正的代理服务器

即：你的请求 -> 代理软件 -> 代理服务器 -> 目标服务器

####  2.2. <a name='port'></a>什么是端口（port）

> 你应该会在 ICS2 学到这一点

网络的流量是通过端口来区分的，所有网络服务会占用（监听）一个端口，比如你连接到服务器的 ssh 服务，本质上是在访问它的 22 端口，你连接 https 网站的本质是在访问它的 443 端口。

反过来，电脑上用来被 ssh 连接的服务就是监听 22 端口，用来被 https 访问的服务就是监听 443 端口。你的本地代理软件也会监听一个端口，比如 7890，10809 等等，你的网络请求会被发送到这个端口，再到代理服务器，再到目标服务器，但是代理软件之后的事情通常是你不用管的。你需要知道的一般只有你的代理软件监听哪个端口

比如默认下，C软件是7890，V软件是10809

如果你的软件是某种奇怪的魔改版，没有告诉你端口是多少，但是它说它用的是“系统代理”，你可以去系统代理到地方看一下它怎么配的（你把整篇都读完就懂这句话的意思了）

但是还有一种代理是TUN模式的，它会假装自己是一张虚拟网卡，此时对于电脑服务来说，走代理是无感的，代理都被隐藏在这个虚拟网卡内部，它对应的操作就很不一样了（比如你要配的就是怎么让你的服务用这个虚拟网卡，而不是别的网卡）

####  2.3. <a name='IP'></a>什么是 IP

IP 就是网络中一个电脑的地址，也叫IP地址，但是一个电脑上有多个网络服务，他们用端口隔离，所以用 `IP:port` 才是对应到某个具体服务的最终地址

比如 `https://10.21.8.35:443` 的 IP 地址是 `10.21.8.35`，端口是 `443`，`https://` 是协议，你应该理解成内容的一部分。

不过通常 https 服务运行（监听）在 443 端口上，所以你们访问网站的时候直接写 `https://10.21.8.35` 就相当于 `https://10.21.8.35:443`，这一步是你用的软件帮你做的（比如浏览器）。

> `10.21.8.35` 是 `go.ruc.edu.cn` 的 IP 地址

一个重要的知识点是 `127.0.0.1` 是本地电脑的地址，你可以理解成你自己的地址，`localhost` 也是同样的意思，即你可以用这个地址访问你自己的电脑

但是我们很少直接输ip，因为记不住，所以网络服务提供者往往会买一个域名，你输入域名后电脑会先询问DNS服务器这个域名对应啥ip，然后用这个ip替代域名访问

DNS服务器一般是你连上网后，比如校园网自动提供的，他会告诉你的电脑一个ip，让你用这个服务器。你也可以自定义，比如8.8.8.8就是谷歌的DNS服务器

因为域名和ip最后都是指向某个电脑主机，所以我们经常用host来统称这两者

比如你可以把localhost当作127.0.0.1的域名，你用 http://127.0.0.1:8080 访问到的内容和 http://localhost:8080 访问到的是一样的

原理是这个域名和ip的关系被写在了本地hosts文件里

电脑里有一个hosts文件，它的作用发生在你查询DNS服务器之前，如果hosts文件里写了某个域名对应的ip是什么，他就会直接用这个ip

所以如果有一些ip你想自己给他一个别名，你就可以写在自己的hosts文件里，或者是服务商刚刚修改某个域名指向的ip，但DNS服务器还没更新，你也可以用hosts来自定义

##### 服务之间的联通性

但不是所有电脑的网络服务直接都是可以直接联通的，无论是他们有没有网络链路相连，还是他们有没有设防火墙，或者某个服务有没有监听某个 ip（服务可以只监听 127.0.0.1，这样只有本地可以访问），都会影响到某个服务能否通信

一般你可以通过 `ping` 指令来测试机器和机器是否联通，比如 `ping 10.21.8.35` 或者 `ping go.ruc.edu.cn` 会告诉你你的电脑和 `go.ruc.edu.cn` 之间的网络链路是否通畅，如果你在校内就可以联通，他会告诉你两个机器之间的网络延迟。

ping还可以顺便用来查某个域名实际的ip是多少（你可以ping个百度试试）

但是服务能不能联通还要看那个服务有没有监听某个ip，比如校园网认证的https服务没有监听 10.21.8.35，那即使 ping 通了，你也访问不了 `https://go.ruc.edu.cn` 网站

##### Windows 和 WSL 的网络

如果你没有改过网络模式，WSL 默认是 NAT 模式，简单说就是 WSL 里的网络是独立的。你可以理解为彼此是独立的电脑，他们有不同的 IP 地址。他们理论上是可以联通的，但是受“IP 之间的联通性”这一节所说的因素影响，他们可能不能直接联通。研究中间哪个环节有问题通常很麻烦，你可以让AI帮你解决这个问题。

但如果你想他们的网络联通，最简单的办法就是改网络模式，让 WSL 的网络变成镜像模式，你可以理解成他们真正意义上是同一个电脑了，他们的端口会互相影响（本来都可以监听 443，现在会冲突），但是好消息是，他们都可以用 `127.0.0.1` 来访问对方了，或者说，访问对方和访问自己是一样的了，而 `127.0.0.1` 通常是所有服务都监听的，总之就是肯定能通的。

####  2.4. <a name='kind'></a>代理的种类

代理的协议有非常多种，在“你的请求 -> 代理软件 -> 代理服务器”这条链路上，“代理软件 -> 代理服务器”之间的连接协议会非常复杂，但是“你的请求 -> 代理软件”的协议通常非常简单，常见的有 http 和 socks5 两种，通常前者兼容性更好，后者支持的功能更多，但不是所有软件都支持 socks5

如果它没说，你应该默认是 http 代理

下文我们只说http代理怎么配，socks5你可以问一下AI

####  2.5. <a name='how'></a>怎么使用代理

一般代理这个功能是由你使用的软件来使用的，更高级的情况见后文。

比如 `curl` 这个命令行工具可以这样走 http 代理访问 `https://example.com`

```
curl -x http://<proxy_ip>:<proxy_port> https://example.com
```

其中 `-x http://<proxy_ip>:<proxy_port>` 就是在访问 http 协议的代理，这个代理的 ip 和端口是 `<proxy_ip>:<proxy_port>`

如果你的代理软件运行在本地（比如你的各种神奇小软件），那么 `<proxy_ip>` 就是 `127.0.0.1`，如果你是在 NAT 模式的 WSL 里，你想走 Windows 的代理，那么 `<proxy_ip>` 就是 Windows 的 ip，获取这个 ip 请问 AI，但正如我们前面所说，用镜像最好，那样就是 `127.0.0.1`

端口就要看那个软件监听的端口了，比如**默认** C 软件是 7890，V 软件是 10809

socks5 我们就不做介绍了，有需要的自查

####  2.6. <a name='what'></a>什么是系统代理

所谓系统代理就是系统级别的全局代理，每个软件可以读取这个信息来配置自己的代理服务（软件也可以选择不读取这个信息，所以这个代理不是绝对的）

比如对于桌面系统，你只要打开软件，打开系统代理就是一个按钮，当然本质是它把它的配置写到你的系统代理里，然后你的浏览器比如 Chrome 就会读这个信息然后走代理了，然后你就可以访问你想访问的网站了。

对于非图形化的系统，比如非桌面版本的 Ubuntu（我们的服务器和WSL通常是这个系统），系统代理的配置方法是通过环境变量，比如你可以在命令行里输入

```bash
export http_proxy=http://<proxy_ip>:<proxy_port>
export https_proxy=http://<proxy_ip>:<proxy_port>
```

这样命令行工具就会通过读 `http_proxy` 和 `https_proxy` 这两个环境变量来配置代理

你可以试一下

```bash
# 假如之前 http_proxy 和 https_proxy 都没有值
curl https://example.com # 超时

export http_proxy=http://<proxy_ip>:<proxy_port>
export https_proxy=http://<proxy_ip>:<proxy_port>
curl https://example.com # 成功
```

你会发现后面就能访问了

**但是，通常系统代理都是指代理 `http` 和 `https` 流量，也就是像访问网页一样的流量，而 ssh 这样的流量是不会走这个代理的，或者说他们不会读取这个信息来走代理，你需要单独配置，见后文**

###  3. <a name='WSL'></a>WSL 怎么配置为镜像模式

默认的 WSL 网络是 NAT 模型，我们要改为镜像模式，即 WSL 的网络和 Windows 的网络是一样的

1. 你需要知道你的用户名

   在 Windows 命令行里输入

   ```
   whoami
   ```

   输出类似

   ```
   tianxuan\17473
   ```

2. 打开文件夹 `C:\Users\<username>`，其中 `<username>` 对应你的用户名，比如 `C:\Users\17473`

3. 查看是否有一个 `.wslconfig` 文件，没有就创建（注意，这个文件没有后缀名，如果你的资源浏览器不显示后缀名，你可以百度一下怎么显示后缀名，程序员建议永远让电脑显示后缀名，因为我们经常要创建没有后缀名或者自定义后缀名的怪东西）

4. 打开它（比如用 VSCode）

5. 添加以下内容，如果 `[wsl2]` 段已经存在就添加到对应的地方

   ```
   [wsl2]
   networkingMode=mirrored
   dnsTunneling=true
   firewall=true
   autoProxy=true
   ```

   > 网络上的古老教程会让你添加到 [experimental] 段，但现在这些选项早就是正式选项了，所以建议直接添加到 `[wsl2]` 段

   具体这些参数的含义可以阅读微软的官方文档：https://learn.microsoft.com/zh-cn/windows/wsl/wsl-config

6. 重启 WSL

   在 Windows 的命令行里输入

   ```
   wsl --shutdown
   wsl
   ```

7. 怎么检测是否配置成功（如果你后面的步骤不成功，你可以检测一下）

   比较简单的办法是让 Windows 启动一个 http 服务（创建一个网站），然后在 WSL 里看看能不能从 `127.0.0.1` 访问，或者反过来可能也算一种测试方法
   
   在被访问端（比如 Windows）上安装 python （Windows 需要安装，WSL 是自带的）

   ```bash
   # 随便位置打开命令行
   python -m http.server 8080
   ```

   WSL 可以执行

   ```bash
   curl http://127.0.0.1:8080
   ```

   如果看到一些 html 代码，那就是成功了

   ```bash
   jarden@TIANXUAN:~$ curl http://127.0.0.1:8080
   <!DOCTYPE HTML>
   <html lang="en">
   <head>
   <meta charset="utf-8">
   <title>Directory listing for /</title>
   </head>
   <body>
   <h1>Directory listing for /</h1>
   <hr>
   <ul>
   ...
   ```

   反过来，如果你是在 WSL 里执行 python 命令，那么你可以在 Windows 的浏览器里直接访问 http://127.0.0.1:8080

###  4. <a name='httpsclonepullfetchG'></a>从 https 协议 clone/pull/fetch G 网站的代码

如果你访问的链接是这样的：`https://github.com/RUCICS/cachelab2.git`，那么你需要尝试这一节的配置方法

一句话：用代理服务的 ip 和端口设置系统代理

如果你完全理解了上文的内容，你就可以自己配置了

简单来说，我们假设你要让 WSL 能够 git clone

1. 设置 WSL 为镜像模式
2. 获取代理软件的端口（ip 因为是本地，所以就是 127.0.0.1）
3. 设置系统代理

   ```bash
   export http_proxy=http://127.0.0.1:<proxy_port>
   export https_proxy=http://127.0.0.1:<proxy_port>
   ```

你可以把

```bash
export http_proxy=http://127.0.0.1:<proxy_port>
export https_proxy=http://127.0.0.1:<proxy_port>
```

写到 `~/.bashrc` 里，这样每次打开终端都会自动跑这两句，否则你每次新打开一个终端，它没有这两个环境变量，你都得输一遍

> 我们不推荐让 WSL 保持 NAT 模式，然后折腾 Windows 的 IP 是多少，怎么连不上的问题了，弄成镜像就是 127.0.0.1，最简单省事

###  5. <a name='sshclonepullfetchG'></a>从 ssh 协议 clone/pull/fetch G 网站的代码

####  5.1. <a name='magic'></a>不配代理的神奇方案（也许未来这个方法会失效）

除了配代理，有一种神奇的方法可以让你不用配代理，通过一个没有神秘电波干扰的 ip 访问 G 网站的代码

假设你是 WSL（Ubuntu），打开 `~/.ssh/config` 文件，没有则创建一个

添加以下内容，如果 `Host github.com` 已经存在，你应该注释掉原有的内容

```
Host github.com
    Hostname ssh.github.com
    Port 443
    User git
```

我们假设你已经配置过这个系统上的 ssh key 了，如果没有，你可以参考 https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh

```bash
ssh -T git@github.com
# 你会看到类似 Hi panjd123! You've successfully authenticated, but GitHub does not provide shell access. 的输出
```

我简单描述一下为什么这样就好了

1. 神秘电波干扰的是 `github.com` 这个域名对应的服务器，所以你访问 G 网站是访问不了的，你 `git clone https://github.com/RUCICS/cachelab2.git` 或者 `git clone git@github.com:RUCICS/cachelab2.git` 也是不行的，因为里面都是 `github.com`，无论你走什么协议，反正最后的那个 ip 都是会被神秘电波拦掉的
2. `git clone git@github.com:RUCICS/cachelab2.git` 的内部是一个 ssh 协议，和我们连服务器用的协议是一样的，只是它连完就给你发了个文件你可以这么理解，所以你可以用 ssh 命令 `ssh -T git@github.com` 来测试连接是否正常（而不是一个 git 命令）
3. 先简单解释一下 `~/.ssh/config` 
   ```bash
   Host github.com            # 用来指示配置对哪个主机名（别名）生效
      Hostname ssh.github.com # 用来指示实际访问的主机名（域名）
      Port 443                # 用来指示访问的端口
      User git                # 用来指示访问的用户名
   ```

   `~/.ssh/config` 文件可以记忆/配置 ssh 的行为，比如你平时连 ics 提供的服务器是

   ```bash
   ssh u2021201626@10.77.110.227 -p 5022
   ```

   你可以在 `~/.ssh/config` 里配置

   ```
   Host ics
      Hostname 10.77.110.227
      Port 5022
      User u2021201626
   ```

   这样上面那堆东西就可以简化成

   ```bash
   ssh ics
   ```

   也就是 ics 变成了一个别名，它会自己找 `~/.ssh/config` 里的配置

   恰巧，github.com 提供了一个奇怪的服务，它的本意我就不赘述了，感兴趣的自己读一下 [Github 文档](https://docs.github.com/zh/authentication/troubleshooting-ssh/using-ssh-over-the-https-port)，简单来说就是除了 `ssh git@github.com`，其实 `ssh git@ssh.github.com -p 443` 也能访问到一样的内容，而神秘电波不知道为什么没有拦截 `ssh.github.com` 这个服务器的流量，所以你相当于访问到了一个 Github 的官方镜像服务器，所有东西都一样，但是神秘电波不知道为什么不干扰（也许是因为这个服务器是专门用来 ssh 的，不是用来访问网页的）

   你可以 ping 一下这两个域名 `github.com` 和 `ssh.github.com`，你会发现前者是不通的，后者是通的
4. 所以最后，每当你 `git clone git@github.com:RUCICS/cachelab2.git` 的时候，它会把 `github.com` 解释为 `~/.ssh/config` 里的一个配置，然后给你走 443 端口连到 ssh.github.com，然后让你的代码恰好都不用动

####  5.2. <a name='proxy'></a>配代理访问的方法  

上面这个方法虽然简单，但总给人不太踏实的感觉，我们还是留一个配代理的方法以备不时之需

因为我们说了，ssh 流量不会走系统代理，所以你需要单独配置 ssh 的代理，这个配置方法和上面的 `~/.ssh/config` 有点像

在 WSL 中打开 `~/.ssh/config` 文件，没有则创建一个

如果是 http 代理，则输入

```
Host github.com
      Hostname github.com
      User git
      ProxyCommand nc -X connect -x proxy_host:proxy_port %h %p
```

其中 `proxy_host` 和 `proxy_port` 是你的代理软件的 ip 和端口，如果是本地代理，那么就是 `127.0.0.1`

> 还是那句话，我们不推荐让 WSL 保持 NAT 模式，然后折腾 Windows 的 IP 是多少，怎么连不上的问题了，弄成镜像就是 127.0.0.1，最简单省事

如果你好奇 `ProxyCommand nc -X connect -x proxy_host:proxy_port %h %p` 是啥意思可以问 AI
