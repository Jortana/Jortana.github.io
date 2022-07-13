---
title: 用docker封印EasyConnect并连接远程桌面和数据库
date: 2021-07-13 23:25:44
categories: 实用工具
tags: 
  - Docker
  - EasyConnect
  - Clash
  - 实用工具
  - Tips
---

我们学校要使用深信服家的 EasyConnect 才能在外面连内网，我又经常需要连学校的电脑和数据库，在使用了一段时间 EasyConnect 之后，我发现这玩意儿不仅流氓，而且难用。流氓在于不用他的时候进程关不掉，卸载还卸载不干净，也不知道他到底要获取什么数据，关于深信服的黑料网上一搜一大堆。难用在于我一开他，虽然是能连上学校内网了，但是我们学校校园网非常差劲，速度本身就不快，还经常连不上 GitHub，用了他之后，我在路由器上配置的代理也没有用，所以我心想能不能抛弃这个流氓客户端，就在不久前，我终于找到了一套解决方案。

<!-- more -->

## 准备工作

这一套解决方案的原理大致就是使用 Docker 运行 EasyConnect 并暴露出一个 sock5 端口作为代理，然后用配置 Clash 的代理规则，按照一定的规则进行转发。一开始是在知乎的一篇文章：[M1 Mac 用不了深信服 easyconnet？ 用 docker+clash封印它](https://zhuanlan.zhihu.com/p/385845245) 中发现了这个方法，然而他的情况和我稍微有点不一样，他没有连接远程桌面等需求，并且写的也比较简略，所以我就萌发了写一篇 **保姆级** 流程介绍的想法。

所以，我们需要用到的软件有：

* Docker
* Clash
* VNC Viewer（如果你用的 EasyConnect 不需要图形界面就不需要）
* 文本编辑器（比如 VS Code）

### 安装软件

#### 1 Docker

不同的平台安装 Docker 有些不一样，Windows 略微麻烦一些，请按照官网的教程（英文）： [Windows](https://docs.docker.com/docker-for-windows/install/)，[MacOS](https://docs.docker.com/docker-for-mac/install/)，或者菜鸟教程（中文）：[Windows](https://www.runoob.com/docker/windows-docker-install.html)，[MacOS](https://www.runoob.com/docker/macos-docker-install.html)一步步进行安装，更推荐官网。

#### 2 Clash

##### Windows

在 Clash For Windows 的 Github 上的 [Release](https://github.com/Fndroid/clash_for_windows_pkg/releases) 页面选择带有 **win** 标识或者后缀为 .exe 的版本点击进行下载，最好选择后缀名为 **.7z** 的压缩包，下载下来后不用安装，解压即可使用。

##### MacOS

同样是在 Clash For Windows 的 Github 上的 [Release](https://github.com/Fndroid/clash_for_windows_pkg/releases) 页面选择相应的版本点击进行下载，不过要选择带有 mac 标识版本，如果你是 M1 芯片的 Mac，请选择带有 **arm64** 标识的版本。

当然你也可以选择 ClashX 等具有相应功能的其他软件。

![选择相应的版本下载](用docker封印EasyConnect并连接远程桌面和数据库/选择相应的版本下载.png)

#### 3 VNC Viewer

如果你用的 EasyConnect **需要** 图形界面登录的话，就需要安装 VNC Viewer，否则可以跳过这一步，如果不知道可以先不装，到后面用 Docker 运行 EasyConnect 的时候提示你不能在命令行界面完成再返回这一步安装。

VNC Viewer 的安装非常简单，到官网的 [下载](https://www.realvnc.com/en/connect/download/viewer/) 页面，选择相应的版本下载，然后一路下一步就完成了。

#### 4 文本编辑器

因为需要编辑一些配置文件，所以需要用到文本编辑器，推荐使用 [VS Code](https://code.visualstudio.com/)，不使用记事本的原因是它不带高亮，没有单词拼写提示也不会自动调整格式，可能会导致你有一些缩进、拼写和标点等方面的错误比较难发现。

## 封印 EasyConnect

### 使用 Docker 运行 EasyConnect

首先不要忘了确定你的 Docker 是正在运行的，使用 Windows 的同学应该可以在右下角看到一只顶着集装箱的小鲸鱼的图标。打开一个命令行/终端，根据你需不需要图形界面输入不同的命令。

**纯命令行版：**

```bash
touch ~/.easyconn
docker run --device /dev/net/tun --cap-add NET_ADMIN -ti -v $HOME/.easyconn:/root/.easyconn -p 127.0.0.1:1080:1080 -e EC_VER=7.6.7 hagb/docker-easyconnect:cli
```

其中 `-e EC_VER=7.6.7` 表示使用 `7.6.7` 版本的 EasyConnect，请根据实际情况修改版本号，根据提示输入服务器地址、登陆凭据。

**图形界面版：**

```bash
docker run --device /dev/net/tun --cap-add NET_ADMIN -ti -e PASSWORD=xxxx -v $HOME/.ecdata:/root -p 127.0.0.1:5901:5901 -p 127.0.0.1:1080:1080 hagb/docker-easyconnect:7.6.7
```

其中 `hagb/docker-easyconnect:7.6.7` 表示使用 `7.6.7` 版本的 EasyConnect，请根据实际情况修改版本号。输入完之后打开 VNC Viewer，点击左上角的 **File -> New connection**，或者在空白处 **右键 -> New connection**，或者使用快捷键 **Ctrl + N**，新建一个链接，在弹出的对话框中的 VNC Server 中填入 `127.0.0.1:5901`，Name 可以不填或者填你喜欢的名称作为一个标签。

![VNC Viewer配置](用docker封印EasyConnect并连接远程桌面和数据库/VNC Viewer配置.png)

点击 OK 后，双击我们刚刚创建好的链接，输入密码 XXXX，就可以进行连接了，进去之后是熟悉的界面，输入地址，用户名和密码之后就可以关掉了。

顺便说一句，在 Docker 中运行的 EasyConnect 都不用输验证码，就更加方便了，由于这个项目本身是会记住你输入的服务器地址的，所以以后使用的时候只需要点一下登录按钮就可以了。

进行到这一步，就会有一个 socks5 代理跑在你电脑的 **1080 端口了**，接下来就是配置 Clash，在我们需要的时候进行转发。

有关于 docker-easyconnect 更详细的配置可以参考 [https://github.com/Hagb/docker-easyconnect](https://github.com/Hagb/docker-easyconnect)。

### 配置 Clash

#### 配置 Clash 代理规则

这一篇只讲如何让你在需要的时候使用 EasyConnect 的 VPN 服务，如果你还需要配置其他代理，可以移步 [官方文档](https://docs.cfw.lbyczf.com/)，不过话说回来，如果你需要配置其他代理，你的机场应该有详细的教程。

首先运行 Clash，点击左侧导航栏的 Profiles，你会看到一个默认的 config.yaml，如果你有其他配置文件也可以编辑其他配置文件。点击你想编辑的配置文件右边的形如 `< >` 的按钮，意思就是在文本编辑器中编辑配置文件，在最后增加：

```yaml
# 增加一个名字是 vpn 的 sock5 代理
proxies:
 - {"name": "vpn", "type": "socks5", "server": "127.0.0.1", "port": "1080"}

# 添加 ip-cidr 的规则
rules:
  - IP-CIDR,222.192.6.0/24,vpn
```

**【注意】**这里填写的 `222.192.6.0/24` 是指只有访问 `222.192.6.xxx` 的流量才会走上面配置的那个代理，需要根据你的实际情况进行更改。当然，你如果是编辑你自己的配置文件，需要在原来的 `proxies` 和 `rules` 下新增，而不能再新增这两个属性。

到这一步，在 Clash 的 General 页面，将 System Proxy 打开，你就已经可以访问学校内网的网页了，但是不可以使用远程桌面等功能，如果需要使用这些功能，还需要进一步配置。

#### 配置 TUN 模式

对于 TUN 模式，官方文档是这么描述的：

> 对于不遵循系统代理的软件，TUN 模式可以接管其流量并交由 CFW 处理，在 Windows 中，TUN 模式性能比 TAP 模式好。

浏览器之类的应用都是使用系统代理的，一些非系统代理应用，可以通过设置被 CFW 接管。总之，配置 TUN 模式之后，我们的需求就可以实现了，启动 TUN 模式需要进行如下操作（MacOS 用户跳过前两步）：

1. 打开 Clash 的 General 页面，找到 Home Directory，点击右边的 `Open Folder` 打开 `Home Directory` 文件夹；

2. 进入网站 [Wintun](https://www.wintun.net/)，点击界面中 `Download Wintun xxx` 下载压缩包。根据系统版本将 `wintun-x.xx/wintun/bin` 对应目录中 `wintun.dll` 复制至 `Home Directory` 目录中。基于 `x64` 的处理器的 `64` 位操作系统请使用 `amd64` 版本，M1 版本的 Mac 选择 `arm64` 版本；

3. 点击 General 中 Service Mode 右边 Manage，在打开窗口中安装服务模式，安装完成应用会自动重启，Service Mode 右边地球图标变为绿色即安装成功；

4. 点击 **settings -> Profile Mixin -> YAML**，点击 **YAML** 右侧的 **Edit**，输入下面的内容，点击右下角的按钮保存。
   
   ```yaml
   mixin: 
     hosts:
       'mtalk.google.com': 108.177.125.188
       'services.googleapis.cn': 74.125.203.94
       'raw.githubusercontent.com': 151.101.76.133
     dns:
       enable: true
       default-nameserver:
         - 223.5.5.5
         - 1.0.0.1
       ipv6: false
       enhanced-mode: redir-host #fake-ip
       nameserver:
         - https://dns.rubyfish.cn/dns-query
         - https://223.5.5.5/dns-query
         - https://dns.pub/dns-query
       fallback:
         - https://1.0.0.1/dns-query
         - https://public.dns.iij.jp/dns-query
         - https://dns.twnic.tw/dns-query
       fallback-filter:
         geoip: true
         ipcidr:
         - 240.0.0.0/4
         - 0.0.0.0/32
         - 127.0.0.1/32
       domain:
         - +.google.com
         - +.facebook.com
         - +.twitter.com
         - +.youtube.com
         - +.xn--ngstr-lra8j.com
         - +.google.cn
         - +.googleapis.cn
         - +.gvt1.com
     tun: 
       enable: true
       stack: gvisor
       dns-hijack:
         - 198.18.0.2:53
       macOS-auto-route: true
       macOS-auto-detect-interface: true # 自动检测出口网卡
   ```

5. 返回 general 页面开启 Mixin 即可，之后点击 connections 会发现所有连接都是 TUN 模式，这时候我们就完成了所有配置

更详细的配置可以参考官方文档的 [TUN 模式](https://docs.cfw.lbyczf.com/contents/tun.html) 页面。

#### 再次启动

当你关掉命令行/终端窗口，或者重启电脑，想再次启动 Docker 时，我们只需要启动相应的容器，并且在 VNC Viewer 中点击登录就可以（如果你不使用图形界面的话只需要启动容器）。

启动容器的操作就不用再去打上面创建容器时的那一串命令了，因为那一串命令的意思是创建容器、下载镜像、启动并进入容器，我们已经创建好了容器，以后只需要启动就行了。所以，我们先要知道我们有哪些容器，然后启动相应的容器就可以。

首先，查看我们已有的容器

```bash
docker ps -a
```

你应该会得到类似的输出，如果你是第一次使用 Docker，应该只有一个容器

![查看容器](用docker封印EasyConnect并连接远程桌面和数据库/查看容器.png)

可以看到，我的第一个容器就是正在使用的，当我重启之后又想开启它的时候，只需要输入

```bash
docker start 81f
```

就可以了，其中 `81f` 是我要启动的容器的 `ID` 也就是第一列的值，这个值很长，通常你只需要输入前几位就可以，或者你也可以记住容器的 `name`，使用

```bash
docker start lucid_allen
```

来启动，容器的名字是可以在创建的时候指定的，也可以通过 `docker rename` 命令修改。

如果你是 Windows 用户，双击右下角 Docker 的图标，你会看到一个可视化的管理界面，点击左侧导航栏的 **Containers / Apps** 也可以看到所有的容器，并且可以通过可视化的方式对容器进行操作。

![可视化操作](用docker封印EasyConnect并连接远程桌面和数据库/可视化操作.png)

在开启容器之后，如果你需要图形界面操作就来到 VNC Viewer 中点击登录，并且开启 Clash 的 System Proxy 就完成了。

## 卸载 EasyConnect

前面说道，EasyConnect 的流氓之处之一就是非常难卸载，但是我还是根据网上的资料和我自己操作的经验总结出一套方法，这一套方法可能略显复杂，可能有多余的操作，不过按照这样操作是可以将 EasyConnect 完全卸载干净的。

首先，找到 EasyConnect 的安装目录，双击运行他的卸载程序，等他运行完之后你会发现，这个目录并没有被完全删掉，还是有很多文件，这时候我们执行下列步骤：

1. 全选 Shift + Delete，如果遇到不给删的就跳过，这时候还是会剩下很多文件。
2. 断网重启电脑，来到安装目录下，把所有文件夹都改名，随便改成什么，再次重启
3. 全选刚刚改过名的文件夹 Shift + Delete，如果这时候还删不了就把文件夹剪切到别的地方再删除
4. 重启电脑，管理员权限打开命令行（如果你没有常用的命令行工具可以 Win + X 选择 **Windows PowerShell（管理员）**）
5. 输入命令 `netsh winsock reset` 回车

最后两步重置 winsock 非常重要，如果不执行很可能会出现各种客户端无法连接网络的情况。
