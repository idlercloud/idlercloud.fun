---
title: "最长公共子序列 (LCS) 在 diff 命令中的应用——Myers 算法"
subtitle: ""
date: 2022-08-11T18:52:26+08:00
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
    - 算法
    - 动态规划
    - C/C++
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
    enable: true
lightgallery: false
seo:
    images: []

repost:
    enable: false
    url: ""
# See details front matter: /theme-documentation-content/#front-matter
---

今天 (2022/2/26) 做 CS 110L 的 week2 作业，内容是用 Rust 编写一个简单的 `diff` 命令。

<!--more-->

我才知道原来 Linux 和 Git 的 `diff` 命令，它的底层算法之一叫做 Myers 算法，而其根本原理是最长公共子序列。

本文讲述 **LCS 的使用和 Myers 算法**。前置知识是动态规划求解 LCS。

<!--more-->

## diff 命令做了什么

如果你常用 GitHub，应该经常可以看到如下界面。

![diff 示例](/images/what_diff_does.png)

这表示 `diff` 命令显示出的本次提交和仓库里之前版本的区别。如你所见，红色、以 `-` 开头的行代表删除，绿色、以 `+` 开头的行代表新内容。

你可能会说，上图这样明显是对同一行进行修改，而并没有删除或者添加行啊。确实，但是仔细想想，修改的操作其实可以等价地转换为删除原来的行，然后添上修改之后的行。

这样的等价转换对于底层算法的实现是比较有利的。

我们把代码不同版本的对比问题抽象一下。首先是代码，代码是一个行构成的序列，也就是说，它的先后关系是重要的；那么，两个版本的代码之间的比较，就相当于两个序列之间的比较。

除去删除或者添加的行，你可以看到还有很多不变的行。它们就是两个序列所共有的成分。而你可以想一下，删除的行，实际上是旧版本代码的特有内容；新添的行，实际上是新版本代码的特有内容。

那么每个代码都是由特有和共有的内容混合组成的，如下。

code1: 共有 1 旧特有 1 共有 2 旧特有 2 旧特有 3 共有 3

code2: 共有 1 共有 2 新特有 1 共有 3 新特有 2

我们要做的，其实就是判断两个代码之间哪些是共有、哪些是旧版本特有，哪些是新版本特有的。

到这你就可以想到了，共有的成分，那不就是公共子序列吗。

## 最长公共子序列(LCS)

这一部分我假定你是有相关基础的，如果没有的话建议去搜几篇文章看看。

我们来回忆一下，LCS 描述这样一种问题：在两个序列中，找到它们共有的最长的子序列。序列一是 `1 2 3 4`，序列二是 `1 3 4 5`，那么 LCS 就是 `1 3 4`。

LCS 的求法，比较常规的是动态规划 $O(n^2)$ 求解。

假定两个序列是 `s1` 和 `s2`，长度分别为 `n` 和 `m`，我们定义二维数组 `dp[n+1][m+1]`（+1 是为了方便）。`dp[i][j]` 表示“`s1` 前 `i` 项和 `s2` 前 `j` 项的 LCS 的长度”。那么就有如下状态转移函数。

$$
\begin{aligned}
dp[i][j]=
\begin{cases}
&0,&若 i=0 或 j=0\\\\
&dp[i-1][j-1]+1,&若s[i-1]=s[j-1]\\\\
&max(dp[i-1][j],dp[i][j-1]),&若s[i-1]\ne s[j-1]
\end{cases}
\end{aligned}
$$

## Myers 算法

如果让你用 LCS 来做 `diff`，你会如何完成？我的第一想法是，从`dp[n][m]`开始，可以依次往上追溯，找到所有的公共项。那么两个序列中剩下的部分就是各自特有的了。

Myers 算法也是这个思路，不过它稍微高明一些，在回溯的过程中就同时判定公共项和特有项。

我们先来看一张图。

![LCS 二维表格](/images/LCS_grid.png)

> 图片来自[这个博客](https://www.cnblogs.com/zqybegin/p/13734107.html)

上图是序列 `ABCBDAB` 和 `BDCABA` 的 dp 结果。箭头表示转移路径。

如你所见，从最右下角出发，沿着路径一直向左上方找到起点（灰色底色）。

Myers 算法的思路就是：如果它是向左上转移，说明这是公共部分；如果向上转移，那么它就是第一个序列的特有部分；如果向左转移，那么就是第二个序列的特有部分。

C++ 代码如下（Rust 直接改的，我没测试）

```cpp
void print_diff(vector<vector<int>> &dp, vector<string> &s1, vector<string> &s2, int i, int j) {
    if(i > 0 && j > 0 && s1[i - 1] == s2[j - 1]) {
        print_diff(dp, s1, s2, i - 1, j - 1);
        cout << "  " << s1[i - 1] << endl;
    } else if(j > 0 && (i == 0 || dp[i][j - 1] >= dp[i - 1][j])) {
        print_diff(dp, s1, s2, i, j - 1);
        cout << "+ " << s2[j - 1] << endl;
    } else if(i > 0 && (j == 0 || dp[i][j - 1] < dp[i - 1][j])) {
        print_diff(dp, s1, s2, i - 1, j);
        cout << "- " << s1[i - 1];
    } else {
        cout << endl;
    }
}
```

你可以看到，它是一个递归的做法，以此来让输出顺序颠倒。

你也可以不用递归，每次把输出结果存在一个 `stack` 中，以此用循环完成。

## 总结

有了以上知识点，想实现 `diff` 就很容易了。我们可以用 `getline` 之类的读入代码，用一个 `vector<string>` 存储，每一项存储一行代码。然后再用一个 LCS 函数求出 `dp` 数组。最后传 到`print_diff` 函数，就可以得到结果了。

不过，LCS 的 DP 求解算法复杂度是 $O(n^2)$，而且据说被证明不可改进。所以据说实践中常常用一些线性复杂度的近似算法。

> 版权声明：本文采用<a rel="license" href="http://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>进行许可，转载请注明出处。
>
> 本文链接：[{{< permalink >}}]({{< permalink >}})
