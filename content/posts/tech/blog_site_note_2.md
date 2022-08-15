---
title: "「个人博客建站笔记」2.网站的布局"
subtitle: ""
date: 2022-08-11T11:21:45+08:00
draft: false
author: "cxz888"
authorLink: "https://cxz888.xyz"
authorEmail: "idlercloud@gmail.com"
description: ""
keywords: ""
license: '<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a>'
# comment: false
weight: 0

tags:
    - 网站
    - WordPress
categories:
    - tech

hiddenFromHomePage: false
hiddenFromSearch: false

summary: ""
resources:
    - name: featured-image
      src: featured-image.jpg
    - name: featured-image-preview
      src: featured-image-preview.jpg

toc:
    enable: true
math:
    enable: false
lightgallery: false
seo:
    images: []

repost:
    enable: false
    url: ""
# See details front matter: /theme-documentation-content/#front-matter
---

本文讲述**如何优化 WordPress 网站的界面布局**。

<!--more-->

-   涉及内容：网站后台、主题、PHP、HTML、CSS。
-   面向人群：新手。
-   前置知识：WordPress 网站的搭建（见[「个人博客建站笔记」——1.网站建成](https://cxz888.xyz/posts/blog_site_note_1)）。

## 网站后台

如果你刚刚搭建好一个 WordPress 网站，那么此时你应该得到了一个默认主题下的主页，只有一个示例页面。

此时网页的功能很简陋，几乎只能写写文章，发发评论。像什么标签、分类、留言板都没有。而且默认的主题可能和你的审美对不上，还有标题字体太小、主页面背景不好等问题。

那么就需要利用网站的后台进行一些自定义的设置了。

点击左上角仪表盘的标志，或者在你网址的后方加上 `/wp-admin`，就可以进入后台。

后台左侧的菜单里有很多选项，这里不一一细讲。

不过值得一提的是，在默认的主题下，你可以在外观-编辑器（beta 版）里可视化地调整页面布局，这一点还是很方便的。外观-自定义是另一个方便的工具，也是可以即时预览你的修改的，你还可以在里面设置网站的名称、副标题和图标。

当然，上述修改大多只是小幅微调，想要有大的整体页面上的变化，需要你进行很多很细致的调整。真正一步到位，改变整个网站的是接下来介绍的主题。

## 主题

如同很多软件一样， WordPress 同样有主题系统。它的主题不仅改变网页的样式，还会更改网页的交互逻辑，比如，如何登录、如何评论等等。

在后台的外观-主题页面里，你可以在线搜索主题，也可以上传一个 zip 包压缩的主题。主题的推荐可以去知乎等地求解。就我所知许多开发者会在 Github 上开源他们的主题，我之前用的 WordPress 主题就是 [Github 上面](https://github.com/dimpurr/Clearision)下载，然后自己做了一些修改。

一些比较完善的主题会提供友好的界面来让你进行一定程度的定制，而无需写代码。如果你没有什么代码基础，那么还是建议仔细挑选一个这样的主题。不过再如何完善，其自由度也很难及得上写代码。

大多数时候，用好主题就可以得到一个令人满意的优美网站，不过如果你还想要更多、更个性化的定制，那么你可以继续往下阅读，我会讲一些前端和后端相关的代码处理。而如果你不想写代码，可以跳到最后，我会推荐一些个人感觉不错的免费主题。

## HTML

一般地，网页本质上都是 HTML 代码构成的。

HTML 中文名称超文本标记语言(Hyper Text Markup Language)，听上去很厉害，其实所谓超文本就是说，它能够表达的内容不单单是文本。

如下代码就表示一个文字内容为“只能按”的按钮。

```html
<button>只能按</button>
```

它真正表现在网页上就是这个样子：<button>只能按</button>。

层层嵌套的 HTML 代码，可以生成功能十分丰富的网页。

比如你现在看的这个网页，你可以在空白处右键，可能就会有“查看网页源代码”的选项。你将会在其中看到整个网页的 HTML 形式，如果你将它复制下来存到本地再打开，完全可以得到一个一模一样的网页。

当然，只是看上去一样，实际上去你去点那些按钮，可能都是毫无反应的，因为网页系统往往是由很多个 HTML 文件构成，而且还会依托其它的一些东西。

比如说，CSS，或称层叠样式表(Cascading Style Sheets)。

## CSS

HTML 描述网站的内容，但它对于内容的样式、形貌的表达力是比较薄弱的。

比如你想要控制刚刚那个按钮，它背景色什么样子、前景色什么样子、字号多大，距离右边框多远等等。HTML 很难完成这些，而这就是 CSS 发光发热的领域了。

> 事实上，你应该很少会需要写 HTML 代码，因为它是主管内容的——而我们有更好的、可视化的方式来管理内容，比如 markdown 或者是 WordPress 自带的编辑器。相对来说你可能会和 CSS 打更多交道，它是主管外貌的。

## PHP

如果按照程序员的分类，HTML 和 CSS 所作的都是“前端”的工作，简单来说就是直接呈现在用户视觉上的效果。

> 其实，传统的前端开发是使用 HTML+CSS+JavaScript 三件套的。不过我对 JS 目前没有太多了解，就先不写了。

而这背后，当然还需要一个工具人，处理网页和服务器之间的数据交换。

PHP 是一个后端开发的语言。

目前我最常用到 PHP 的场景，其实是需要做条件判断的时候。比如用户是否登录，肯定是要展现不一样的界面的；还有就是根据权限隐藏掉一部分不太好展示的东西。

## 例子：Clearision

Clearision 是之前本站的主要主题。它相对来说功能比较简陋，但是界面我蛮喜欢的。

而且功能简陋，所以代码逻辑也比较好理解、好修改。

所以接下来我就以它为例子，讲一讲怎么自定义主题。

### 主题提供的设置

正如大部分主题，Clearision 在后台提供了一些设置方便自定义网页的外观和交互。如下：

![Clearision 设置](/images/clearision_settings.png)

在这里你可以设置主页上的 Logo 头像、网页的图标、社交主页的网址、是否显示访客环境、是否显示作者等。

整体而言功能不多。

### 修改主题文件

接下来是比较关键的部分。在 Clearision 设置的下方，可以看到主题文件编辑器。

![Clearision 自定义主题](/images/clearision_custom.png)

是的，接下来就需要修改代码了。

首先要提一点的是，Wordpress 官方推荐使用子主题的方式来自己修改主题，这样的好处是如果原主题更新了，不会覆盖掉自定义的内容。方法可以在 wordpress 的教程里找到。

对于 Clearision 没有这个必要，一来是原主题已经很久没更新过了，估计以后也不会更新了，二来是子主题的方案还是会导致一些问题（如缓存等）。

我们直接在原文件里更改就行。也就是上图右侧那些主题文件。有些文件的中文会指出它的用处。

这些文件的加载顺序、逻辑结构也是很有讲究的，这里不深究，有兴趣可以去自行搜索。

`functions.php` 和 `style.css` 是我经常打交道的两个。

`functions.php` 添加一些功能性的操作。比如说我想修改用户注销之后跳转的界面，就可以在里面加入如下内容

```php
# 注销后重定向至
add_action('wp_logout','auto_redirect_after_logout');
function auto_redirect_after_logout(){
	nocache_headers();
	wp_safe_redirect( home_url() );
	exit();
}
```

它也可以用来修改一部分的外观内容，主要是是否展示，而不管其细节。

而正如上面所言，`style.css` 主管整个网站的所有样式细节。

比如我想修改用户“发表评论”按键的样式，就可以这么写。

```css
/* 发表评论的样式 */
#cmt_submit {
    margin: 0;
    -moz-box-sizing: border-box;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    -webkit-border-radius: 0;
    border-radius: 0;
    background: #6f6f6f;
    color: #eee;
    font-weight: 600;
    display: inline-block;
    border: none;
    cursor: pointer;
    -webkit-border-radius: 3px;
    border-radius: 3px;
}
```

语法部分我不讲，去找些教程有点基本的了解即可。

另外，CSS 中是可以重复定义一个元素（部件）的样式的。Clearision 原作者在 CSS 里面写了很多内容，如果想要修改怎么办？无需找到相关的内容然后修改，只要在最下方新添样式就可以了。后定义的样式会覆盖先定义的样式。

你可能在 CSS 编辑界面上方看到提示，说可以在“内建的 CSS 编辑器”中修改 CSS。

这个实际上就是我上面提到的外观-自定义里面的一个小功能，所谓的额外 CSS。界面大致如下：

![额外 CSS](/images/extra_css.png)

在左侧的文本框中输入的 CSS 代码，会实时地同步到右侧的网页中，对于调整外观来说十分方便。

这个功能配上浏览器提供的“检查”（审核元素、开发者工具、F12）会如虎添翼。只需在想要修改样式的地方右键->检查，就会出现类似下面的界面（这是 Edge 浏览器的界面）

![开发者工具](/images/edge_f12.png)

在右侧的开发者工具中就可以查看网页的 HTML 代码和 CSS 样式，甚至可以直接在里面修改。

你可能会说了，不会写代码怎么办？其实我也不是很会，CSS 和 PHP 我都没有系统学习过。大部分的代码其实都来自于网络，只要可以合理描述自己的需求，还是能够找到想要的内容的。

但是完全不会写代码还是很不便的。网上的代码质量参差不齐，而且可能有过期风险，也可能面临不同浏览器、不同屏幕分辨率等一系列问题。复制代码之后可能会还需要一些微调来解决这些问题。所以有更高需求的话建议还是沉下心来细细学习一下，总之不会亏。

## 主题推荐

下面是我用过的，比较适合个人博客的两款主题，都是简约风的。

就算你打算自定义主题，我也推荐你下载几个美观的主题看看。它们会给你增添灵感，而且你可以参考它们的代码实现。

### MDX

这是我个人比较喜欢的另一个主题。功能丰富，界面简洁美观。如果我最先接触到它的话，也许就会使用它。

[主题主页](https://mdx.flyhigher.top/)，[Github 仓库](https://github.com/yrccondor/mdx)

主题主页里有详细的展示，我建议你可以稍微看一下。下面我也会放一些本站使用该主题后的样貌。

首页：

![mdx 首页](/images/mdx_home_page.png)

文章：

![mdx 首页](/images/mdx_passage.png)

这里我没有仔细去调整，实际上，在后台中有非常多的主题样式搭配。如图。

![mdx 设置](/images/mdx_settings.png)

在我短暂的使用中，有两个功能让我很喜欢。

一个是开启夜间模式，另一个是目录。

现在的这个主题已经支持这两个特性了，很好用。（本站已经不是用 WordPress 而是 Hugo 了）

### Indite

这个主题可以直接在 Wordpress 后台里添加。

大致外观如下：

![indite 外观](/images/indite_appearance.png)

可以看到配色比较柔和，页面也很简洁。

这个主题的大部分功能都可以通过外观-自定义修改。不过里面的设置都是英文的，可能需要花些力气。

另外一点不太好的是，这个边栏太大了，影响阅读体验。

## 总结

这就是本文的所有内容了。断断续续拖了一个月才完成，着实有些难写。

想写的内容其实还有很多，但是难以表述出来。如果有疑问的话欢迎评论区里提问。

> 版权声明：本文采用 [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/) 进行许可，转载请注明出处。
>
> 本文链接：[https://cxz888.xyz/posts/blog_site_note_2/](https://cxz888.xyz/posts/blog_site_note_2/)
