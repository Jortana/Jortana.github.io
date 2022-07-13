---
title: Plex 加载本地 nfo 文件和本地外挂字幕
date: 2021-07-30 07:01:31
categories: 服务器应用
tags:
  - Plex
  - 家庭影院系统
  - 家用服务器
---

前几天在鼓捣 Plex 的时候吃了一记 [tinyMediaManager](https://www.tinymediamanager.org/) 的安利，虽然他现在有一些收费的选项，不过还是有曲线救国的方法，可以免除搜索上限的方法（其实就是下别人弄好的包）。tMM 的好处在于你可以清楚地看到每部电影、电视剧的各种信息，是没找到还是漏了一些或者全找到了，一目了然，但是当 Plex 导入资料库时，往往不会用你搜集好的信息和图片，可能会自己再去搜一次，搜的不一定对，浪费时间，如果出现要手工调整的情况操作起来也比 tMM 麻烦一些，如何让 Plex 自动识别本地的信息呢，需要利用到 Plex 的插件：XBMCnfoMoviesImporter 和 XBMCnfoTVImporter。

<!-- more -->

要实现上述功能只需要两大步：加载插件和配置插件。

## 1 加载插件

### 1.1 下载插件

这里提供三个下载方式，其中包含的文件都是一样的，GitHub 链接、百度网盘和我的私有网盘。如果网盘都失效了而且 GitHub链接出现问题，可以自行去 GitHub上搜索这两个插件。

GitHub：

​	XBMCnfoMoviesImporter：[仓库地址](https://github.com/gboudreau/XBMCnfoMoviesImporter.bundle) [下载地址](https://github.com/gboudreau/XBMCnfoMoviesImporter.bundle/archive/master.zip)

​	XBMCnfoTVImporter：[仓库地址](https://github.com/gboudreau/XBMCnfoTVImporter.bundle) [下载地址](https://github.com/gboudreau/XBMCnfoTVImporter.bundle/archive/master.zip)

[百度网盘](https://pan.baidu.com/s/1-1xVafu0Pt7XZVlovIWFqg) 提取码：mict

[我的网盘](http://jortana.tech:8000/s/SzQEwcFt5sikjq7) 提取码：RsKkjXGBRW

### 1.2 安装插件

下载完成后解压，你会得到两个文件夹，如果你是从 GitHub 上下载，注意更改一下文件夹的名字，确保这两个文件夹的名字是 **XBMCnfoMoviesImporter.bundle** 和 **XBMCnfoTVImporter.bundle**。然后将这两个文件夹放到 Plex 的插件目录下，在官网的 [说明文档](https://support.plex.tv/articles/202915258-where-is-the-plex-media-server-data-directory-located/) 中可以找到你的平台对应的目录。

我是在 Ubuntu Server 中安装的，所以我的目录为 `/var/lib/plexmediaserver/Library/Application Support/Plex Media Server/Plug-ins/` ，注意后面多了一个 `/Plug-ins/`，官网给的目录是 Plex 的安装目录，我们要把插件复制到 **插件文件夹** 里去。

将两个文件夹都丢进去之后，我们重启 Plex 服务，Linux 用户可以使用命令 `sudo service plexmediaserver restart `，接下来进行一下插件的配置。

## 2 配置插件

### 2.1 使 Plex 加载本地信息

说是配置插件，实际上是配置资料库的代理。进入 Plex 的控制面板，点击设置，左边的导航条拉到最下面，选择 **管理** 下的 **资料库**，找到你需要操作的资料库点击 **编辑资料库**。

在编辑面板中选择 **高级**，并更改代理，如果是电影资料库就将代理改为 XBMCnfoMoviesImporter，如果是电视节目资料库就将代理改为 XBMCnfoTVImporter。

根据网上的经验，最好关闭 Enable generating Collections from tags 这一选项（只有电影资料库有）并将 Collections 选项调整为 Disabled。

![更改代理](Plex-加载本地-nfo-文件和本地外挂字幕/image-20210730235320150.png)

### 使 Plex 加载本地字幕

到这一步，插件已经可以帮我们加载本地的影片信息了，但是还不能自动加载本地外挂字幕，即使字幕的命名没有问题，Plex 也不能识别，还需要对插件本身进行配置。

从左边导航栏找到 **设置** 下的 **代理**，在右边的电影和电视节目中可以看到我们刚刚加载的插件，将插件下的 Local Media Assets **勾选上**。

![加载本地字幕](Plex-加载本地-nfo-文件和本地外挂字幕/image-20210731000245536.png)

这时候，刷新资料库的元数据，你会发现所有的信息包括海报、缩略图、字幕等全都已经按照本地的文件加载好了。

