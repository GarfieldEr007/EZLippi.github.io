---
layout: post
title: Git魔法 - Git大师技巧
keywords: gitmagic, git, 技巧, 高级技巧
category : git
tags : [Git魔法, git, gitmagic]
---
## Git大师技巧 ##

到现在，你应该有能力查阅 *git help* 页，并理解几乎所有东西。然而，查明解决特
定问题需要的确切命令可能是乏味的。或许我可以省你点功夫：以下是我过去曾经需要
的一些食谱。

### 源码发布 ###

就我的项目而言，Git完全跟踪了我想打包并发布给用户的文件。创建一个源码包，我运
行：

    $ git archive --format#tar --prefix#proj-1.2.3/ HEAD

### 提交变更  ###

对特定项目而言，告诉Git你增加，删除和重命名了一些文件很麻烦。而键入如下命令会容易的多：

    $ git add .
    $ git add -u

Git将查找当前目录的文件并自己算出具体的情况。除了用第二个add命令，如果你也打
算这时提交，可以运行`git commit -a`。关于如何指定应被忽略的文件，参见 *git help ignore* 。

你也可以用一行命令完成以上任务：

    $ git ls-files -d -m -o -z | xargs -0 git update-index --add --remove

这里 *-z* 和 *-0* 选项可以消除包含特殊字符的文件名引起的不良副作用。注意这个
命令也添加应被忽略的文件，这时你可能需要加上 `-x` 或 `-X` 选项。

### 我的提交太大了！ ###

是不是忽视提交太久了？痴迷地编码，直到现在才想起有源码控制工具这回事？提交一
系列不相关的变更，因为那是你的风格？

别担心，运行：

    $ git add -p

为你做的每次修改，Git将展示给你变动的代码，并询问该变动是否应是下一次提交的一
部分。回答“y”或者“n”。也有其他选项，比如延迟决定；键入“？”来学习更多。

一旦你满意，键入

    $ git commit

来精确地提交你所选择的变更（阶段变更）。确信你没加上 *-a* 选项，否则Git将提交
所有修改。

如果你修改了许多地方的许多文件怎么办？一个一个地查看变更令人沮丧，心态麻木。
这种情况下，使用 *git add -i* ， 它的界面不是很直观，但更灵活。敲几个键，你可
以一次决定阶段或非阶段性提交几个文件，或查看并只选择特定文件的变更。作为另一
种选择，你还可以运行 *git commit --interactive* ，这个命令会在你操作完后自动
进行提交。

### 索引：Git的中转区域 ###

当目前为止，我们已经忽略Git著名的'索引‘概念，但现在我们必须面对它，以解释上
面发生的。索引是一个临时中转区。Git很少在你的项目和它的历史之间直接倒腾数据。
通常，Git先写数据到索引，然后拷贝索引中的数据到最终目的地。

例如， *commit -a* 实际上是一个两步过程。第一步把每个追踪文件当前状态的快照放
到索引中。第二步永久记录索引中的快照。 没有 *-a* 的提交只执行第二步，并且只在
运行不知何故改变索引的命令才有意义，比如 *git add* 。

通常我们可以忽略索引并假装从历史中直接读并直接写。在这个情况下，我们希望更好
地控制，因此我们操作索引。我们放我们变更的一些的快照到索引中，而不是所有的，
然后永久地记录这个小心操纵的快照。

### 别丢了你的HEAD ###

HEAD好似一个游标，通常指向最新提交，随最新提交向前移动。一些Git命令让你来移动
它。 例如：

    $ git reset HEAD~3

将立即向回移动HEAD三个提交。这样所有Git命令都表现得好似你没有做那最后三个提交，
然而你的文件保持在现在的状态。具体应用参见帮助页。

但如何回到将来呢？过去的提交对将来一无所知。

如果你有原先Head的SHA1值，那么：

    $ git reset 1b6d

但假设你从来没有记下呢？别担心，像这些命令，Git保存原先的Head为一个叫
ORGI_HEAD的标记，你可以安全体面的返回：

    $ git reset ORIG_HEAD

### HEAD捕猎 ###

或许ORG_HEAD不够。或许你刚认识到你犯了个历史性的错误，你需要回到一个早已忘记
分支上一个远古的提交。

默认，Git保存一个提交至少两星期，即使你命令Git摧毁该提交所在的分支。难点是找
到相应的哈希值。你可以查看在.git/objects里所有的哈希值并尝试找到你期望的。但
有一个更容易的办法。

Git把算出的提交哈希值记录在“.git/logs”。这个子目录引用包括所有分支上所有活
动的历史，同时文件HEAD显示它曾经有过的所有哈希值。后者可用来发现分支上一些不
小心丢掉提交的哈希值。

The reflog command provides a friendly interface to these log files. Try

命令reflog为访问这些日志文件提供友好的接口，试试

    $ git reflog

而不是从reflog拷贝粘贴哈希值，试一下：

    $ git checkout "@{10 minutes ago}"

或者捡出后五次访问过的提交，通过：

    $ git checkout "@{5}"

更多内容参见 *git help rev-parse* 的``Specifying Revisions''部分。

你或许期望去为已删除的提交设置一个更长的保存周期。例如：

    $ git config gc.pruneexpire "30 days"

意思是一个被删除的提交会在删除30天后，且运行 *git gc* 以后，被永久丢弃。

你或许还想关掉 *git gc* 的自动运行：

    $ git config gc.auto 0

在这种情况下提交将只在你手工运行 *git gc* 的情况下才永久删除。

### 基于Git构建 ###

依照真正的UNIX风格设计，Git允许其易于用作其他程序的底层组件，比如图形界面，
Web界面，可选择的命令行界面，补丁管理工具，导入和转换工具等等。实际上，一些
Git命令它们自己就是站在巨人肩膀上的脚本。通过一点修补，你可以定制Git适应你的
偏好。

一个简单的技巧是，用Git内建alias命令来缩短你最常使用命令：

    $ git config --global alias.co checkout
    $ git config --global --get-regexp alias  # 显示当前别名
    alias.co checkout
    $ git co foo                              # 和“git checkout foo”一样

另一个技巧，在提示符或窗口标题上打印当前分支。调用：

    $ git symbolic-ref HEAD

显示当前分支名。在实际应用中，你可能最想去掉“refs/heads/”并忽略错误：

    $ git symbolic-ref HEAD 2> /dev/null | cut -b 12-

子目录 +contrib+ 是一个基于Git工具的宝库。它们中的一些时时会被提升为官方命令。
在Debian和Ubuntu，这个目录位于 +/usr/share/doc/git-core/contrib+ 。

一个受欢迎的居民是 +workdir/git-new-workdir+ 。通过聪明的符号链接，这个脚本创
建一个新的工作目录，其历史与原来的仓库共享：

    $ git-new-workdir an/existing/repo new/directory

这个新的目录和其中的文件可被视为一个克隆，除了既然历史是共享的，两者的树自动
保持同步。不必合并，推入或拉出。

### 大胆的特技 ###

这些天，Git使得用户意外摧毁数据变得更困难。但如若你知道你在做什么，你可以突破
为通用命令所设的防卫保障。

*Checkout*：未提交的变更会导致捡出失败。销毁你的变更，并无论如何都checkout一
 个指定的提交，使用强制标记：

    $ git checkout -f HEAD^

另外，如果你为捡出指定特别路径，那就没有安全检查了。提供的路径将被不加提示地
覆盖。如你使用这种方式的检出，要小心。

*Reset*: 如有未提交变更重置也会失败。强制其通过，运行：

    $ git reset --hard 1b6d

*Branch*: 引起变更丢失的分支删除会失败。强制删除，键入：

    $ git branch -D dead_branch  # instead of -d

类似，通过移动试图覆盖分支，如果随之而来有数据丢失，也会失败。强制移动分支，键入：

    $ git branch -M source target  # 而不是 -m

不像checkout和重置，这两个命令延迟数据销毁。这个变更仍然存储在.git的子目录里，
并且可以通过恢复.git/logs里的相应哈希值获取（参见上面 上面“HEAD猎捕”）。默
认情况下，这些数据会保存至少两星期。

*Clean*: 一些Git命令拒绝执行，因为它们担心会重装未纳入管理的文件。如果你确信
 所有未纳入管理的文件都是消耗，那就无情地删除它们，使用：

    $ git clean -f -d

下次，那个讨厌的命令就会工作！

### 阻止坏提交 ###

愚蠢的错误污染我的仓库。最可怕的是由于忘记 *git add* 而引起的文件丢失。较小
的罪过是行末追加空格并引起合并冲突：尽管危害少，我希望浙西永远不要出现在公开
记录里。

不过我购买了傻瓜保险，通过使用一个_钩子_来提醒我这些问题：

    $ cd .git/hooks
    $ cp pre-commit.sample pre-commit  # 对旧版本Git，先运行chmod +x

现在Git放弃提交，如果检测到无用的空格或未解决的合并冲突。

对本文档，我最终添加以下到 *pre-commit* 钩子的前面，来防止缺魂儿的事：

    if git ls-files -o | grep '\.txt$'; then
        echo FAIL! Untracked .txt files.
        exit 1
    fi

几个git操作支持钩子；参见 *git help hooks* 。我们早先激活了作为例子的
*post-update* 钩子，当讨论基于HTTP的Git的时候。无论head何时移动，这个钩子都会
运行。例子脚本post-update更新Git在基于Git并不知晓的传输协议，诸如HTTP，通讯时
所需的文件。
