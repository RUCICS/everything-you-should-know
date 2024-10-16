# SSH
SSH（Secure Shell) 主要用于远程登录服务器以及执行命令行操作，也可以用来传输文件、端口转发等。SSH 使用加密技术确保数据传输的安全性，防止中间人攻击、数据篡改和窃听。
## SSH密钥配置
密钥工作机制: 密钥分为公钥和私钥. 本地和远程分别持有私钥和公钥，通过算法进行匹配，匹配成功则允许链接。 

## 本地机器生成密钥

```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
执行后会显示，这个路径就是你当前ssh的密钥的路径
```
Enter file in which to save the key (/home/your_user/.ssh/id_rsa):
```
先查看生成的公钥内容：
```
cat ~/.ssh/id_rsa.pub
```

补充说明: 本地机器和远程机器是一个相对的概念。 默认本地机器指你的本地PC。 但是当两个服务器之间配置SSH的时候，你也可以认为一个机器是本地（通常是你权限更高机器） ，另一个就是远程。 
## Github SSH密钥配置
- 登录到你的 GitHub 账户。
- 点击右上角的头像，然后选择 **Settings**（设置）。
- 在左侧菜单中，选择 **SSH and GPG keys**。
- 点击 **New SSH key** 或 **Add SSH key**。
- 在 **Title** 字段中输入一个描述（例如：“My Laptop Key”）。
- 在 **Key** 字段中粘贴你之前复制的公钥。
- 点击 **Add SSH key** 按钮。

测试Github的SSH连接：
```bash
ssh -T git@github.com
```


FAQ1： 为什么powershell里面可以clone但是wsl里面不能clone
> 因为两个系统存储私钥的位置不同。 解决方案是把win下面的私钥复制到wsl系统的`~/.ssh/id_rsa`即可

FAQ2： 私钥的权限设置
```shell
chmod 600 ~/.ssh/id_rsa
```

## Visual Studio SSH Config配置
1. 安装remote ssh插件
2. 打开ssh config文件
3. vscode ssh config的基本格式
```
Host myserver
    HostName server_ip
    User username
    Port 1234 
    IdentityFile ~/.ssh/id_rsa
```
## 一般的远程服务器免密码配置
远程服务器上放公钥，本地服务器放私钥
```
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "你的公钥内容" >> ~/.ssh/authorized_keys # 注意 你应该将你的公钥内容替换！ 
chmod 600 ~/.ssh/authorized_keys
```
## 终端连接软件: Terminus
> Connect with one click from any mobile and desktop device. No re-entering IP addresses, ports, and passwords.

Terminus的学生认证包含所有必要的功能。 
Terminus一个推荐的本地可以使用的连接远程服务器的可视化软件，可以**配合**VSCode使用。 Terminus支持【文件传输】有可视化的进度条，断点重连，拖拽上传下载等功能。 
Terminus是跨平台工具，可以在多个设备上共享登陆，迁移服务配置。 

## 文件传输：SCP和SFTP协议
`scp`（Secure Copy Protocol）是用于通过SSH在本地和远程服务器之间安全地复制文件的命令。
从本地服务器复制到远程服务器:
```bash
scp -P port_number /path/to/local/file username@remote_host:/path/to/remote/directory
```
从远程服务器复制到本地服务器:
```bash
scp -P port_number username@remote_host:/path/to/remote/file /path/to/local/directory
```
执行完这行命令之后，需要输入密码。 如果已经配置好密钥则不需要。 

SFTP（**SSH File Transfer Protocol** 或 **Secure File Transfer Protocol**）是一种通过 **SSH（Secure Shell）** 连接来提供安全文件传输的协议。它与传统的 FTP（File Transfer Protocol）不同，SFTP 是完全通过加密的 SSH 通道进行通信的，因此在文件传输过程中具有更高的安全性。Terminus就支持基于SFTP的文件传输。 

# Linux系统常见的Shell 命令
https://github.com/RUCICS/everything-you-should-know/blob/main/shell/README.md
# 文件系统
文件系统是计算机上组织文件结构的系统。 Linux系统上暴露给用户的文件组织形式和windows系统上基本一致，都是树状结构。 但是对文件的操作方式从点击变成了命令。 如果了解QT开发或者其他应用开发的话，应该了解过信号-槽函数概念。 举个例子，鼠标在一个按钮上的点击(作为一个信号)会触发一个响应函数执行特定功能。 在Linux系统上，需要使用者直接指出你需要的函数。  

## vim 编辑器
`vi filename.txt` 打开文件
`esc -> :wq enter` 保存并且推出

## 终端复用器Tmux 
- 更新apt:```sudo apt update```
- 安装```apt install tmux```

## 当你遇到没有提及的问题
1. 相当一部分命令在报错的同时会给出解决方案，因此请仔细阅读报错信息
2. 在详细的给出上下文的情况下，LLM几乎可以给出解决方案
3. 如果以上两条都不能解决问题，请原始报错+尝试过的方案及其他附加信息寻求助教帮助
# zsh:  功能更加强大的shell
配置一个现代化的shell可以帮助大家在写命令行程序的时候节省时间。 
```bash
sudo apt install zsh
# 使用which zsh 确认zsh路径，也可能是 /usr/bin/zsh
chsh -s /bin/zsh # chsh for change shell 
```
### Oh my  zsh 插件
oh my zsh 官网  https://ohmyz.sh/  安装:
```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
通过gitee镜像安装: 
```
sh -c "$(curl -fsSL https://gitee.com/Devkings/oh_my_zsh_install/raw/master/install.sh)"
```
重新加载终端来配置这个插件
## Powerlevel10k主题
powerlevel10K: https://github.com/romkatv/powerlevel10k
```
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ~/powerlevel10k
echo 'source ~/powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
```
## 设置conda环境
```shell
conda init zsh 
conda init 
conda activate base 
```
## zsh插件

### 自动补全: zsh-autosuggestions

clone github仓库
```shell
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```
或者国内gitee镜像
```shell
git clone https://gitee.com/liangjuda/zsh-autosuggestions.git $ZSH_CUSTOM/plugins/zsh-autosuggestions
```
然后vim打开zsh的配置文件
```
vim ~/.zshrc
```
找到plugins这一行，如果没有可以自行添加
```
plugins=(git zsh-autosuggestions)
```
添加完成之后保存退出，运行
```
source ~/.zshrc
```
### 高亮: zsh-autosuggestions 
clone github仓库
```shell
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```
或者国内gitee镜像
```shell
git clone https://gitee.com/testbook/zsh-syntax-highlighting.git  $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
```

然后vim打开zsh的配置文件
```
vim ~/.zshrc
```
找到plugins这一行，如果没有可以自行添加
```
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```
添加完成之后保存退出，运行
```
source ~/.zshrc
```
### 插件补充说明
关于添加插件的一些补充说明: 
1. 添加方式： clone仓库，并且在配置文件里面加入插件名字，然后重新运行配置文件
2. 添加插件可能会导致shell反应变慢，因此插件并不是越多越好
3. 关闭再打开shell后修改才会生效