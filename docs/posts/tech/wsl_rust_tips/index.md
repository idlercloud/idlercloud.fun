# 使用 WSL2 时碰到的小问题


写这篇博客时，我因为 CS110L 的作业要求必须安装 Linux 环境。几番周折最后选择了 Windows 下的 WSL2。现在看来很不错。

过程中也遇到一些小坑：

<!--more-->

1. 磁盘占用问题，`wsl --install`默认安装在 C 盘
2. 代理问题 "# Failed to establish a socket connection to proxies: ["PROXY XXX.XXX.XXX.XXX:7890"]"
3. 换源问题，源和系统版本不一致导致升级的包不对
4. Rust 编译报错"/usr/bin/ld: cannot find Scrt1.o: No such file or directory"

下面一一详细记录了问题和解决方案，参考了很多网上的方法，都附了链接。

## 磁盘占用问题

安装 WSL2，刚开始我用的是微软文档里写的方法，直接在命令行里`wsl --install`。后来我忽然意识到，这样安装的系统默认是在 C 盘的。而我的 C 盘早都红了，只留下 7、8 个 G 的样子。考虑到未来的使用恐怕是不足够的。

好嘛，只好先卸载 WSL 了。准确来说是卸载 WSL 里面安装好的 Linux 发行版。这个找网上教程即可。

> 顺便一提，如果不用`wsl --install`，按照网上找到的教程，你需要手动去控制面板打开 Windows 的 WSL 选项。重启后再去手动下载 ubuntu 发行版。`wsl --install`应该是帮你把这两步都做了，但是还是需要重启的，它会在重启后才安装 Linux 发行版。

想要自定义安装路径，就需要手动下载 ubuntu 安装包，在[微软 WSL2 文档](https://docs.microsoft.com/zh-cn/windows/wsl/install-manual) 中查看发行版列表。如下

![微软 Linux 发行版列表](/images/microsoft_linux_distribution.png)

如果本文附带的链接失效了，你可以自己去搜索微软的 WSL2 文档。

点击上图中你想要的发行版，就会开始下载。另外提一句，最上面的 Ubuntu 似乎是 20.04 版本的，不知道它和下面的 Ubuntu 20.04 有什么区别。

下载好的文件大概是这个样子。

![捆绑包示例](/images/downloaded_bundle_example.png)

直接运行它会直接通过 Microsoft Store 安装，不过我们不这么做（因为可能改不了路径）。我们把它当作压缩文件打开，比如改后缀打开，或者右键、打开方式里选压缩软件打开。内容大致如下。

![捆绑包内部](/images/ubuntu_bundle_inner.png)

里面有很多东西，不过比较大的就一个 x64 结尾，一个 ARM64 结尾。根据架构自行选择即可。

把这个 `.appx` 文件当作压缩文件解压。如果不行，就先把它后缀的改成 `.zip` 再解压。解压的目标位置就是你希望 Linux 发行版安装的位置。

现在你应该得到了一堆文件，其中应该有一个很显眼的 `ubuntu.exe`。运行它，等待安装就可以了。最后它可能会生成一个 `.vhdx` 文件，这应该就是你的 Linux 发行版系统内部所有文件的存储位置了（所以其它文件可以删了）。

最后，在命令行输入 `wsl`，就可以启动 Linux 系统啦。

参考文章：[自定义 WSL 的安装位置，别再装到 C 盘啦](https://zhuanlan.zhihu.com/p/263089007?ivk_sa=1024320u)

## 代理问题

如果你挂了梯子，那么可能在 WSL 里遇到问题。你可能会看到类似于 `# Failed to establish a socket connection to proxies: ["PROXY XXX.XXX.XXX.XXX:XXXX"]` 这样的错误。而且你会发现，在 Windows 上能访问 Google 等网站，在 WSL 上用 `wget` 等命令就无法访问。

此时可以先去 powershell（注意在宿主机，也就是 Windows 上）执行 `ipconfig` 查看 WSL 的 IPv4 地址。类似于下图

![WSL 的 IP 地址](/images/WSL_ipaddress.png)

假设你查询到的 IP 地址是 XXX.XXX.XXX.XXX，那么就在 WSL 上修改 `http_proxy` 和 `https_proxy` 这两个环境变量。比如用`export`命令。

```shell
export http_proxy="http://XXX.XXX.XXX.XXX:PORT"
export https_proxy="http://XXX.XXX.XXX.XXX:PORT"
```

其中 PORT 是你的代理软件开放的 LAN 端口，对于 clash 而言是 7890。（2022/8/11 时）

注意 `https_proxy` 的值不需要是 `https`，否则未来很可能在使用 `curl` 时出现 OpenSSL 错误。

这个命令是每次打开 WSL 都要输的，你可以用别名来简化流程，也可以设置 /etc/profile 文件来一劳永逸。

而且你还需要打开你代理软件的“允许 LAN”的选项。如果你用的是 Clash for Windows，就是首页的那个 Allow LAN。

如果现在还不行，有可能是防火墙的问题，在控制面板->系统和安全->Windows Defender 防火墙->允许应用通过 Window 防火墙中，把你的代理软件的相关项全部打上勾。

现在在 WSL 里应该也可以访问 Google 了。

如果某一天你发现 WSL 里面代理又不好使了，可以重新在 Windows 下输入 `ipconfig` 查看 WSL 的 IP 地址。这个地址不是固定的，有可能变化。

参考博客：[WSL 开发环境的坑（不定期更新）](https://blog.lishunyang.com/2021/02/wsl2-dev-env.html)

## 换源问题

换源也不是随便换的。

首先用命令 `lsb_release -a`，查看自己的发行版本和代号。

比如我用的这个是 ubuntu 20.04 代号为 `focal`。

那么相应的，换源时就得注意这个代号。

首先用 `sudo vim /etc/apt/sources.list` 打开文件（如果你不会使用 vim 就用别的文本编辑器）

然后删除原来的内容，输入

```
#阿里源
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
```

注意看，其中有 `focal` 这个代号，一定要和自己版本的代号一致。

换完源之后别忘了

```powershell
sudo apt-get update    # 更新源
sudo apt-get upgrade   # 更新软件包
```

参考博客：[Ubuntu 20.04 && Ubuntu 18.04 修改 apt 源](https://blog.csdn.net/WU2629409421perfect/article/details/110881141)

## rust 编译报错

```
/usr/bin/ld: cannot find Scrt1.o: No such file or directory
/usr/bin/ld: cannot find crti.o: No such file or directory
```

这个可能是因为我刚开始换源没换对引发的错误。

总之如果你出现这个问题，可以尝试按上面操作换到正确的源上，然后更新一下试试。

如果不行的话，再使用 `sudo apt-get install libc6-dev` 安装软件包。这样应该能解决问题了。

[usr/bin/ld: cannot find crti.o: No such file or directory](https://blog.csdn.net/weixin_42255281/article/details/110820663)

## 总结

WSL2 要占用不少内存，不过使用起来体验比 VMWare 要好很多。如果你不是很在乎图形界面的话，那么推荐可以尝试一下。（其实也有在 WSL2 中使用 GUI 的方法，不过似乎要 Win11）

> 版权声明：本文采用 [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/) 进行许可，转载请注明出处。
>
> 本文链接：[{{< permalink >}}]({{< permalink >}})


---

> 作者: <no value>  
> URL: http://idlercloud.xyz/posts/tech/wsl_rust_tips/  

