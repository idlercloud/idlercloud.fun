---
title: "在 Rust 中表征树形结构"
subtitle: ""
date: 2022-08-14T23:42:42+08:00
draft: true
author: "cxz888"
authorLink: "https://cxz888.xyz"
authorEmail: "idlercloud@gmail.com"
description: ""
keywords: ""
license: '<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a>'
weight: 0

tags:
    - Rust
    - 数据结构
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

树形结构在软件中相当常用。比如文件目录、右键菜单、多级收藏夹等。

在其它语言，如 C/C++ 中，表达树形结构可能仅需要指针即可，然而在 Rust 中需要稍费周折。

<!--more-->

![树示例](/images/tree_eg_content.png "Hugo 文章结构")

众所周知，Rust 中想要实现一个足够通用的双向链表并不是容易事，其根本原因是所有权机制。

在链表这样底层的数据结构中，所有权关系往往并没有那么明确。对于树来说也是同样。

下面我来尝试梳理一些解决方案。

## 应用场景

首先来假定一个简单的应用场景吧。

![Acrobat 书签](/images/adobe_acrobat_csapp_catalog.png "Acrobat 书签，CSAPP 目录")

假设我们希望实现 Acrobat 这样的多级书签功能，并且需要可以修改：包括增删改书签、调整顺序、调整级别。

这是一个广义的非二叉树结构。或者更准确点是个森林结构，因为顶级标签并无父节点，不过这无伤大雅。

总之抽象出来的需求就是：

1. 除根节点外，每个节点具有一个父节点
2. 每个节点可以具有多个子节点，也可以没有
3. 节点具有数据值（在这里是标签名）

## 递归数据结构

我们用 `Item` 代表每一个标签项。

一个优美而非常具有诱惑力的想法是朴素的递归数据结构：

```rust
struct Item {
    name: String,
    children: Vec<Item>,
}
```

> 顺便一提，如果希望树是异构的，比如需要在类型系统上区分内节点和叶节点，可以使用 `enum`

非常简单。而如果希望遍历整个树，代码也非常简洁：

```rust
impl Item {
    fn walk(&self) {
        println!("{}", self.name);
        for child in &self.children {
            child.walk();
        }
    }
}
```

这已经是几乎不能再短的代码量了。

然而这种简洁背后其实隐含的信息是：每个 `Item` 完整拥有其子节点的所有权。

或者说，任何一个子节点，都被看作是父节点的一部分。这意味一旦发生借用，将会影响到整个父节点。

比如说我需要根据节点的名称找到该节点，函数的签名可能类似于：

```rust
fn look_for(&self, name: &str) -> Option<&Item> {
    todo!()
}
```

这里返回的 `Item` 的引用可能需要暂存起来，之后再使用来做某些事。然而既然是引用，就仍然要服从 Rust 的所有权机制。

也就是说，这时候从根开始的整个树是不可变的。这显然极大地妨碍了可进行的操作。

更何况，有时候我们需要返回特定节点的可变引用，以待进行操作，如进行重命名等。

以书签为例，我们可能需要在发生鼠标事件时选中某个标签。也就是返回其引用。而直到很长一段时间之后，我可能忽然需要将它重命名，那么先前返回的就得是可变引用。

而在这段时间之内，就连不可变借用都无法发生。

<img src="/images/tree_error_0.png" style="zoom:80%;">

更糟糕的是，假如我需要调整某个标签的位置呢？如果是在同一层调整先后顺序，那就必须要获取父节点的可变引用，调整其 `children` 字段。

这几乎是不可完成的。所以需要换个思路。

### 变通方法

在换思路之前我们先来看看有没有一些（不优雅的）修补方法。

上述方法之所以不可用，是因为试图用引用来访问某个特定的节点。

既然如此，干脆不引用得了。最极端的方法：每次需要进行操作我都从根节点朝下遍历找到需要的节点。

在节点较少时，这种方案还算可以接受，毕竟代码比较容易写。

另一种稍微优化的方案是，对于每个节点，使用从根到其路径来代表它。

具体而言就是，用一个 `Vec<usize>` 记录路径，表示“根节点的第 x 个子节点的第 y 个子节点的...”。

如下述结构，其中 `life` 节点可以用 `[1,0]` 表示，代表它是根节点 `content` 的第 1 个子节点 `categories` 的第 0 个子节点。

```
content
├── about.md
└── categories
   ├── life
   └── tech
```

在大部分情况下，树形结构的深度都不会太夸张，那么这个方案也是可以接受的。

不过需要注意的是，路径随时可能失效。比如我有一天忽然删除了 `about.md` 这个节点，那么 `categories` 就变成了 `content` 的第 0 个子节点了。

##

> 版权声明：本文采用 [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/) 进行许可，转载请注明出处。
>
> 本文链接：[{{< permalink >}}]({{< permalink >}})
