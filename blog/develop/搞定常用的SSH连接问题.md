---
slug: ssh_connect
title: 搞定常用的SSH连接问题
date: 2023-11-27
authors: kuizuo
# tags: [随笔, code, backup]
# keywords: [随笔, code, backup]
---



## `SSH`解决什么问题

SSH（Secure Shell）主要解决了在不安全网络上进行安全远程登录和其他安全网络服务的问题。它提供了一种加密的通道，使得用户可以安全地远程登录到另一个计算机系统，并且在远程系统上执行命令、传输文件等操作，而无需担心信息被截获或篡改。SSH协议通过对传输的数据进行加密和身份验证，确保了远程连接的安全性。

## 生成`SSH`公钥

命令行输入：`ssh-keygen -t rsa -C "ntscshen@163.com"`

`ntscshen@163.com` 这个可以换成任意的邮箱地址

结果展示如下：

```jsx
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/eguchi/.ssh/id_rsa): # <输入Enter键>
Created directory '/Users/eguchi/.ssh'.
Enter passphrase (empty for no passphrase): # <输入验证密码>
Enter same passphrase again: # <再输入一次相同的验证密码>
Your identification has been saved in /Users/eguchi/.ssh/id_rsa.
Your public key has been saved in /Users/eguchi/.ssh/id_rsa.pub.
The key fingerprint is:
57:15:3c:ca:f2:dc:27:6d:c2:9a:88:d0:70:cf:8d:31 xxx@163.com
The key's randomart image is:
+--[ RSA 2048]----+
|             .o. |
|             .o  |
|           ... . |
|      . . E.o    |
|       +So.O o . |
|      . ..+ + = +|
|       . . . o = |
|        . . o    |
|                 |
+-----------------+
```

如果是第一次生成，默认会在相应路径下`~/.ssh`生成`id_rsa`和`id_rsa.pub`两个文件

复制id_rsa.pub文件信息`cat ~/.ssh/id_rsa.pub`。将这些信息复制到Github的Add SSH key页面即可。

```jsx
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDkkJvxyDVh9a+zH1f7ZQq/JEI79dVjDSG
4RzttQwfK+sgWEr0aAgfnxdxQeDKxIxqI1SwyTY8oCcWzvpORuPqwbc7UWWPcCvbQ3jlEdN
5jvwKM82hincEWwI3wzcnVg2Mn8dH86b5m6REDzwRgozQ3lqrgwGVlTvkHDFs6H0b/1PSrM
XGppOP/QXGEVhZ6Hy4m3b1wMjjrbYwmWIeYklgoGHyrldhAaDYc33y7aUcRyFyq5DubtsLn
2oj4K+1q36iviCHxCOri0FDmn2dzylRCI4S+A2/P7Y7rVfdT+8OWYKCBUs8lfjujghEtejq
Qmj9ikyGTEAW1zQCN778hVwYdjL ntscshen@163.com
```

## **不同的操作系统，均有一些命令，直接将SSH key从文件拷贝到粘贴板中**

1. mac：`pbcopy < ~/.ssh/id_rsa.pub`
2. windows：`clip < ~/.ssh/id_rsa.pub`
3. linux：

```jsx
udo apt-get install xclip
# Downloads and installs xclip. If you don't have `apt-get`, you might need to use another installer (like `yum`)
xclip -sel clip < ~/.ssh/id_rsa.pub
# Copies the contents of the id_rsa.pub file to your clipboard
```

## 连接测试

命令行输入：  `ssh -T git@github.com`，只要见到 `successfully` 就成功了

让SSH走443端口[解决方案](https://help.github.com/articles/using-ssh-over-the-https-port/)

## 重新生成并更新了服务器公钥，连接不上怎么办？

如果你重新生成并添加SSH。出现连接失败场景，你需要更新 `known_hosts` 文件。

你可以将`known_hosts`文件视为SSH的一种缓存机制，用于存储之前连接过的SSH服务器的公钥信息。如果SSH服务器的公钥发生更改（例如，服务器重新安装或SSH密钥对重新生成），你需要更新你的`known_hosts`文件以包含新的服务器公钥。

可以使用 `cat ~/.ssh/known_hosts` 查看里面的记录，我的记录如下

```jsx
github.com ssh-ed25519 AAAAC3Nza...
...
git.xxx.com ssh-ed25519 AAAAC3NzaC1l...
...
124.111.78.185 ssh-ed25519 AAAAC3N...
...
```

解决这个问题最常见的做法是使用`ssh-keygen -R hostname`命令从`known_hosts`文件中删除对于的主机的记录。`hostname` 对应就是 `github.com`、`git.xxx.com`、`124.111.78.185` ，删除成功之后，重新连接服务即可

- `known_hosts`文件是SSH安全基础设施的重要组成部分，它有助于保护SSH连接免受未经授权的访问和篡改。

    `known_hosts`这个文件存储了你连接过的所有SSH服务器的公钥信息。当你尝试连接到一个新的SSH服务器时，你的SSH客户端会检查该服务器的公钥是否在`known_hosts`文件中。如果不在，你的SSH客户端会提示你确认该服务器的公钥，并将其添加到`known_hosts`文件中。如果服务器的公钥已经存在于文件中，但与你尝试连接的服务器的公钥不匹配，你的SSH客户端会拒绝连接，以防止中间人攻击。

## 管理多个SSH Key

使用场景：多场景ssh连接

1. github一个ssh
2. gitlab一个ssh
3. 1个私有的git服务一个ssh
4. 内网一个ssh

### 步骤1

生成的逻辑和第一个一样 `ssh-keygen -t rsa -C "注释内容，一般为邮件地址 xxx@163.com"`

注意此处不要一路回车,否则邮箱将覆盖上一次生成的ssh key,要给这个文件起个名字,例如 `id_rsa_GitLab` 、`id_rsa_tencent_cloud`、… ,默认的是 `id_rsa`

### 步骤2(方案一)

在SSH config文件中定义密钥：打开你的SSH config文件（通常位于`~/.ssh/config`），为每个密钥创建一个新的Host配置。在每个Host配置中，使用`IdentityFile`选项指定对应的私钥文件路径。

```jsx
$ vim ~/.ssh/config
Host sshtest
    HostName ssh.test.com 
    User user
    Port 2200
    IdentityFile ~/.ssh/id_rsa_GitLab

Host ssttest2
    HostName ssh.test2.com
    User user2
    Port 2345
    IdentityFile ~/.ssh/id_rsa_tencent_cloud

Host alias1                          # 机器别名
    HostName 192.168.1.2             # 主机地址
    User root                        # 用户名
    IdentityFile ~/.ssh/id_ecdsa     # 认证文件
    Port 22

Host 服务器名B
    user 用户名
    hostname 服务器ip
    port 端口号
    identityfile 本地私钥地址
```

**常用的Config配置选项**

- 必须配置
  - `Host`：指定配置块
  - `User`：指定登录用户
  - `Hostname`：指定服务器地址，通常用`ip`地址
  - `Port`：指定端口号，默认值为`22`
- 可选
  - `Identityfile`：指定本地认证私钥地址
  - `ForwardAgent yes`：允许`ssh-agent`转发
  - `IdentitiesOnly`：指定`ssh`是否仅使用配置文件或命令行指定的私钥文件进行认证。值为`yes`或`no`，默认为`no`，该情况可在`ssh-agent`提供了太多的认证文件时使用
  - `IdentityFile`：指定认证私钥文件
  - `StrictHostKeyChecking`：有`3`种选项
    - `ask`：默认值，第一次连接陌生服务器时提示是否添加，同时如果远程服务器公钥改变时拒绝连接
    - `yes`：不会自动添加服务器公钥到`~/.ssh/known_hosts`中，同时如果远程服务器公钥改变时拒绝连接
    - `no`：自动增加新的主机键到`~/.ssh/known_hosts`中

### 步骤2(方案二)

我通常使用 `ssh-add` 这样一个命令行工具去管理密钥。

```bash
cd ~/.ssh
ssh-add id_rsa_github
ssh-add id_rsa_tencent_cloud
ssh-add id_rsa_GitLab
# 根据需要，可以添加多个私钥文件
```

添加完成后，使用`ssh-add -l`命令查看代理中的私钥列表

```bash
4096 SHA256:Nb2DeVNxejTquFX7VmDwoLEk******************* github (RSA)
4096 SHA256:2QE7pqZDSntmYk1CVwNyUwNE******************* cloud.tencent (RSA)
```

显示上述内容即代表添加成功

- 这里的代理指的是`SSH`代理，也称为`ssh-agent`。`SSH`代理是一个在后台运行的进程，用于管理和存储`SSH`私钥，以便在`SSH`连接时自动提供它们，而无需用户手动输入密码。

    `SSH`代理通常会在用户登录时自动启动，并一直运行在后台。当用户需要进行`SSH`连接时，`SSH`客户端会向代理请求相应的私钥进行身份验证。代理会验证用户的请求，并在验证通过后提供私钥给`SSH`客户端使用。这样，用户就可以在不需要每次都手动输入密码的情况下进行`SSH`连接了。

### 持久化问题

**`ssh-add` 是用于将私钥添加到 `ssh-agent` 的缓存中，但它并不是用来永久性地存储私钥的**。实际上，它的作用只是把你指定的私钥添加到 `ssh-agent` 所管理的一个 session 当中。而 `ssh-agent` 是一个用于存储私钥的临时性的 session 服务，也就是说当你重启之后，`ssh-agent` 服务也就重置了。所以，如果你希望在重启后仍然能够使用 `ssh-agent` 缓存的私钥，你需要在每次登录后重新运行 `ssh-add` 命令将私钥添加到缓存中。

对于 Linux 系统，你可以将 `ssh-add` 命令添加到你的 shell 启动脚本中，例如 `~/.bashrc` 或 `~/.bash_profile`。编辑该脚本，将 `ssh-add` 命令添加到文件的末尾，并保存。这样，每次你登录时，启动脚本会自动运行并添加私钥到 `ssh-agent`。

打开当前 **shell** 的 rc 文件，我这里使用的是 `.zshrc`，所以我需要打开 ~/.zshrc，在最下面添加两行命令：

```bash
vi ~/.zshrc

# ...
ssh-add id_rsa_github
ssh-add id_rsa_tencent_cloud
```

这样每次运行终端都会自动加载这两个密钥文件

```bash
Identity added: .ssh/id_rsa_tencent_cloud (cloud.tencent)
Identity added: .ssh/id_rsa_github (ntscshen@163.com)
```

我们来将这段每次运行终端自动提示的信息去除掉

```bash
vi ~/.zshrc

...
nohup ssh-add id_rsa_github >> /dev/null
nohup ssh-add id_rsa_tencent_cloud >> /dev/null
```

解释一下执行意思：在后台运行`ssh-add`命令，将`.ssh/id_rsa_github`私钥添加到SSH代理中，并将任何输出丢弃。更详细信息，看下面。

- `**rc**`是“runcom”的缩写，这个术语最初来源于Unix系统，在早期的系统中使用了一个叫做“runcom”的目录，用于存放启动时需要运行的一些脚本和配置文件。后来这个术语被广泛应用于各种类型的系统和软件中，如Linux、BSD、Vim等。“runcom”意味着“运行命令”，这也反映了rc文件的作用，即在程序运行前初始化环境或配置，以便程序能够正常工作。在Linux中，文件名后缀为rc通常表示这是一个shell脚本文件，用来初始化或配置某个程序或系统的环境。

    `**nohup ssh-add id_rsa_github >> /dev/null**`这段代码是一个在Unix和Linux系统中使用的命令，它的主要作用是添加SSH私钥到SSH代理，并且确保命令在后台运行，即使关闭终端也不会停止。下面我将逐一解释代码中的每个部分：

    1. `nohup`: 这是一个Unix和Linux命令，用于运行另一个命令在后台，并且忽略所有挂起的信号。这意味着即使你关闭了运行该命令的终端，命令也会继续运行。
    2. `ssh-add`: 这是用于添加SSH私钥到SSH代理的命令。SSH代理是一个存储私钥的程序，它允许用户进行SSH连接而无需每次都输入密码。
    3. `.ssh/id_rsa_github`: 这是SSH私钥的文件路径。它通常位于用户的家目录下的`.ssh`目录中。这个特定的文件名`id_rsa_github`表明它可能是用于连接到GitHub的私钥。
    4. `>>`: 这是一个重定向操作符，它将命令的输出追加到一个文件中，而不是覆盖文件的内容。
    5. `/dev/null`: 这是一个特殊的设备文件，在Unix和Linux系统中，任何写入`/dev/null`的数据都会被丢弃，读取它会立即返回文件结束(EOF)。在这里，它被用作一个“黑洞”，以丢弃`ssh-add`命令的输出。
