# Linux 通过 gogs 或 gitea 自建 Git 服务


之前写课设时因为需要合作，于是在服务器上搭建了一个临时、简陋的 git 服务。

没有 web 界面，纯命令行交互，添加密钥都要手动 ssh 到服务器。

这次就尝试用开源应用搭建一个功能完善、使用方便、颜值能打的 git 服务。

<!--more-->

## gogs

[官网地址](https://gogs.io/)

国人大佬维护的、以 Golang 开发、支持 Linux、macOS、Windows 和基于 ARM 的操作系统的易用 Git 服务。

它的优点是所需性能极低，甚至可以在树莓派上运行起来；使用极其简单，只需几步设置。

预览（图片来自官网）

![gogs 预览](/images/gogs_preview.png)

### 通过 Docker 使用

这里假设你已经安装好 docker，并且懂得基本的使用。

```bash
# 拉取官方镜像
docker pull gogs/gogs

# 本地创建存放数据的目录
mkdir -p /var/gogs

# 运行镜像
docker run --name=gogs -p 10022:22 -p 10880:3000 -v /var/gogs:/data gogs/gogs
```

`docker run` 中的两个 `-p` 分别代表着 ssh 服务和主服务，将容器内的 22 号端口和 3000 号端口分别映射到外界的 10022 和 10880 端口。

注意，如果你看了一些过期的教程，它们可能让你将 3000 映射到 10080。从前这是可以的，但是某个版本之后，Chrome 因为安全原因禁止了 10080 端口的访问（Edge 和 Firefox 应该也是）。如果你用 10080，可能就会看到 `error: kex_exchange_identification: client sent invalid protocol identifier` 的错误提示。

如果一切顺利，现在就可以通过 `服务器 IP:10880` 在浏览器中看到安装界面了。如下：

![gogs 安装界面 1](/images/gogs_installation_1.png)

数据库可以选 MySQL、PostgreSQL 或 SQLite3。注意如果用 MySQL 需要 5.7 及以上版本。实际上大部分情况 SQLite3 就够用了。

然后是下面的应用基本设置。

一般情况下其它不动，尤其是运行系统用户，似乎必须写 git，因为 docker 的运行脚本里面好像硬编码了 RUN_USER 为 git。

下图红框框选出的三个最好自己决定。应用名称我不知道会影响什么，但域名和应用 url，正如图中提示所言，会影响 clone 的地址。

![gogs 安装界面 2](/images/gogs_installation_2.png)

如果不加修改，最后仓库的 clone 地址就会类似这样：

![gogs 安装界面 3](/images/gogs_installation_3.png)
![gogs 安装界面 4](/images/gogs_installation_4.png)

下面还有可选设置，如邮件服务器、禁止用户注册等等，不一一介绍。

现在，点击“立即安装”即可。

如果没有在可选设置里面注册管理员帐号，那么默认第一个注册的用户即是管理员。

gogs 的一些设置可以在 `数据目录/gogs/conf/app.ini` 修改，如：

![gogs 设置](/images/gogs_settings.png "app.ini 位置")

设置的名称和格式都比较清楚，如有疑问可以查看[仓库里的 `app.ini`](https://github.com/gogs/gogs/blob/main/conf/app.ini)。这是内嵌到二进制分发中的默认设置，其中有每个选项的英文介绍。

### 二进制安装

和 Docker 大同小异。这里参考一下[官方文档](https://gogs.io/docs/installation/install_from_binary)。

根据自己的版本下载二进制文件，上传到服务器。比如我放在了网站目录，实际上随便放个地方都行。

然后这里有个需要注意的点，平时我们 ssh 到服务器可能一直都是用的 root 用户。

而如果你运行 `./gogs web` 时用的是 root 用户，应用设置里填的运行系统用户却是 git。那么最后就会报错 `运行系统用户非当前用户:git -> root`。

实际上 git 服务也完全不需要 root 用户，我们应该切换到普通用户。

最好是用 git 这个用户名，这样最后 ssh 的 clone 地址就是 `git@xxxx`。不过如果你已经安装过 git，那么 git 这个用户可能已经被注册了，而且如果用 `su git` 切换到这个用户后似乎无法正常使用命令。

git 这个用户不行的话，就用 `sudo adduser` 另外创建用户，比如 `sudo adduser gogs`。按照指示完成创建，最后用 `su gogs` 切换过去即可。

cd 到二进制文件所在的目录，如果直接执行 `./gogs web` 可能显示权限不足，那么就先 `chmod +x gogs`。如果是用 SQLite3 的话，为了后续能创建数据库，这里还要进行一下 `chown gogs:gogs /www/wwwroot/repo.cxz888.xyz` 把目录的权限赋予给当前用户。

剩下的就和 docker 差不多了，不过最后 ssh clone 地址是 `gogs@xxxx`。

可能遇到的情况是，gogs 默认情况下使用的是 3000 端口。如果你的 3000 端口已经被占用，那可能就需要一些修改。

按照官方所说，你不应该修改源代码中的 `app.ini`，而是应该在二进制文件所在目录下新建 `custom/conf/app.ini`。（docker 方式安装无需担心这点）

你可以添加如下内容修改端口：

```
[server]
HTTP_PORT        = 10880
```

## gitea

[官网地址](https://github.com/go-gitea/gitea)

gitea 是 2016 年的时候从 gogs 项目中 fork 出来的。

似乎是策略不同，gitea 明显活跃程度更高一些，无论是 issue、PR 还是 commit 都远超 gogs。

我个人也觉得文档、功能、提示方面 gitea 做得更好。（仅个人感受，无意冒犯）

我直接尝试了二进制安装，参考[官方教程](https://docs.gitea.io/zh-cn/install-from-binary/)。

同样需要把目录权限赋予当前用户。

如果在 root 用户下启用 `./gitea web`，它会提示你 git 不需要用 root 用户并退出，硬要用 root 似乎也可以，按它的提示即可。

端口默认也是 3000，如有冲突解决方法和 gogs 类似。

官方提供了一个[样例](https://github.com/go-gitea/gitea/blob/main/custom/conf/app.example.ini)来解释你可以设置的选项。

启动之后的设置和 gogs 基本一样，不过你可以看见，默认主题是暗色主题（我挺喜欢这个）

预览一下最终界面：

![gitea 预览](/images/gitea_preview.png)

## IP 和端口映射

到现在为止，无论是 gogs 还是 gitea，也无论如何安装，用户都得通过 `服务器 IP:端口` 的方式访问主页，这无疑是很不便而丑陋的。

所以应当通过映射来使用域名直接访问。

可以参考我的[「个人博客建站笔记」1.网站建成](http://blog.cxz888.xyz/posts/blog_site_note_1/)中域名解析的部分。

在你的域名服务上提供的 DNS 解析里添加 A 记录。比如我是将 `repo.cxz888.xyz` 解析到自己的服务器 IP。

DNS 解析对端口是一无所知的，所以现在得通过 `repo.cxz888.xyz:端口号` 来访问 git 服务，还是比较丑。

下一步就是通过宝塔的反向代理来解析端口。如果你不是用宝塔，那么可以根据你使用的 web 服务器软件，如 nginx、apache，去百度或谷歌搜索反向代理的方法。

> 你也可以顺手给这个网站申请个 SSL 证书

![宝塔设置反向代理](/images/gitea_bt_reverse_proxy.png)

如上图，在宝塔后台设置反向代理，名称随意填，目标 URL 代表你实际要访问的地址，可以填入 `http://服务器IP:端口号`，发送域名就填上刚刚解析的域名即可。

现在，应该可以直接通过域名进入 git 服务了。

恭喜你完成了自建 git 服务。_★,°_:.☆(￣ ▽ ￣)/$:_.°★_ 。

如有疑问，欢迎评论留言~

> 版权声明：本文采用 [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/) 进行许可，转载请注明出处。
>
> 本文链接：[{{< permalink >}}]({{< permalink >}})


---

> 作者: <no value>  
> URL: http://blog.cxz888.xyz/posts/tech/self-build_git_service_with_gitea/  

