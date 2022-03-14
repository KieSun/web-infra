# Git 工作原理浅析

Git 想必各位读者都会用吧，当然就算你不会用，笔者也会在后续的文章中介绍一些有用的操作。

那么不知道各位在使用 Git 的时候有没有想过这玩意到底是如何做到信息存储的？接下来笔者会对这部分的内容做一个简单的介绍。

首先确定一个概念：在 Git 中万物皆对象，你可以认为 Git 就是一个对象的数据库。后续笔者会使用一些 Git 命令来揭开核心的工作原理。

现在我们需要创建一个 Git 仓库：

```shell
git init test
# Initialized empty Git repository in /Users/yck/github/test/.git/
```

创建完成后我们应该能发现这个文件夹中出现了 `.git` 目录，目录结构如下：

![Ccou3i](https://yck-1254263422.cos.ap-shanghai.myqcloud.com/uPic/Ccou3i.png)

在这个目录中最核心的是 `objects` 目录，虽然目前内部还没有重要的信息。

接下来我们在仓库中随便写点什么东西：

```shell
echo "111" > text.txt
```

此时需要使用一个大家平时应该没有接触过的命令 `git hash-object`

```shell
git hash-object -w text.txt
# 58c9bdf9d017fcd178dc8c073cbfcbb7ff240d6c
```

这个命令能帮助我们根据内容计算出一个 hash 值。如果大家之前创建的文件名及输入的内容与笔者的一致的话，那么输出的 hash 值应该也是一致的，Git 的哈希计算是基于 [SHA-1 算法](https://www.wikiwand.com/zh-sg/SHA-1)。

现在我们查看刚才的 `objects` 目录的话，会发现其中多了一个文件夹：

![RZtyTT](https://yck-1254263422.cos.ap-shanghai.myqcloud.com/uPic/RZtyTT.png)

如果我们把文件夹名和文件名合起来的话会发现这个值刚好等于刚才我们生成的哈希值。此时我们可以推断出一个结论：**`objects` 文件夹中存储的数据会以哈希值的前两位作为文件夹名，后三十八位作为文件名。**

那么这个以 `c9` 开头的文件内部到底存储了什么数据呢？我们用 `cat` 指令查看一下先：

![eZwNeY](https://yck-1254263422.cos.ap-shanghai.myqcloud.com/uPic/eZwNeY.png)

居然是一个乱码的输出。那么我们如何才能查看到正确的数据呢？此时又可以使用一个平时大家接触不到的命令 `git cat-file`

![3YVmir](https://yck-1254263422.cos.ap-shanghai.myqcloud.com/uPic/3YVmir.jpg)

使用这个命令后我们会发现输出的内容与刚才我们创建的文件内容一致，也就是说实际上这个 `c9` 开头的文件内部将信息通过一定的方式存储了起来。另外我们还通过 `-t` 发现了这个对象的类型是 `blob`，这个是我们第一个了解的 Git 对象类型。`blob` 这个概念应该挺多读者都熟，就是一个文件对象，内部以二进制存储数据，不清楚的读者可以阅读 [MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)。

此时如果我们再次修改刚才的文本文件并查看数据：

![JwcY2x](https://yck-1254263422.cos.ap-shanghai.myqcloud.com/uPic/JwcY2x.png)

可以发现 `objects` 目录中又多了一个文件，其中存储的数据依旧对应着当前文本文件中的内容。

**其实这里与笔者先前的认知不同。之前一直以为 Git 中存储的是每次差量的数据，但实际上却是直接把整个文件内容存储了下来，这也就导致对于维护时间长的大型项目来说，`.git` 目录中的内容会很庞大。因为其中记录了每次版本变更涉及到的所有文件数据。**

现在我们能根据上文总结出一些信息：**文件内容会以 `blob` 对象类型存储在 `objects` 文件夹中，并且每次版本变更都是以全量的方式存储数据。**