---
layout: post
title: 用git rebase合并commit
category : Git
---
最近时常用git来作版本控制和代码提交，渐渐发现了git rebase的用法和好处

尤其在整理与合并git的提交记录时，git rebase使用起来十分灵活

### 使用情景
进行一次提交之后，马上发现代码里可能有一两行需要改动，更改之后再次提交，然后又觉得哪里不妥。。。

{% highlight console %}
[root@addcp gitdemo]# vim test.txt
[root@addcp gitdemo]# git commit -am 'add test.txt'
发现一些代码需要小改动
[root@addcp gitdemo]# vim test.txt
[root@addcp gitdemo]# git commit -am 'update test.txt'
...
...
第N次发现
[root@addcp gitdemo]# vim test.txt
[root@addcp gitdemo]# git commit -am 'update test.txt again and again'
...
{% endhighlight %}

这样多次之后，git的commit记录就会变得很乱，一堆小改动，却占用了很多次提交记录：

### 问题重现

{% highlight console %}
[root@addcp gitdemo]# git log
commit abe26fe0f5b1788e8f7b1949082e1477c5337aa0
Author: admin <admin@addcp.com>
Date:   Sat Sep 7 01:29:48 2013 +0800

    update test.txt again and again

commit c3633d5153bacabeceaeaeb552484e7f9b6a2b9c
Author: admin <admin@addcp.com>
Date:   Sat Sep 7 01:29:21 2013 +0800

    update test.txt again

commit c8bc032daac7d0859a0e73cd83a464cee0f59625
Author: admin <admin@addcp.com>
Date:   Sat Sep 7 01:28:51 2013 +0800

    update test.txt

commit 0e80061e90503189dd7f189297aed809953e5e8b
Author: admin <admin@addcp.com>
Date:   Sat Sep 7 01:28:22 2013 +0800

    add test.txt
......
{% endhighlight %}

### rebase 示例及用法

这时如果使用git rebase来解决这个问题就非常容易了

git rebase语法是这样的

{% highlight console %}
git rebase [--interactive | -i] [-v] [--force-rebase | -f] [--no-ff] [--onto <newbase>] [<upstream>|--root] [<branch>] [--quiet | -q]
{% endhighlight %}

这里我们使用到的是`-i`选项，也就是交互的rebase

{% highlight console %}
[root@addcp gitdemo]# git -i HEAD~3
{% endhighlight %}

其中`HEAD~3`是该分支的前三次提交。也可以在后面加`<branch>`的参数指定要衍合的分支。

{% highlight text %}
pick c8bc032 update test.txt
f c3633d5 update test.txt again
f abe26fe update test.txt again and again

# Rebase 0e80061..abe26fe onto 0e80061
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#
# If you remove a line here THAT COMMIT WILL BE LOST.
# However, if you remove everything, the rebase will be aborted.
{% endhighlight %}

其中，`pick`是将该次提交原样提交

`reword`也是提交，不过在提交之前会让你再次修改该次提交日志

`edit`和reword差不多，但是在修改提交日志后，会等待你再次`git commit --amend`，也就是等待你追加一些需要提交的文件与改动。之后再使用`git rebase --continue`完成整个rebase

`squash`是将该次提交合并到前一次提交里，但会将这次提交的日志内容合并到上一次里

`fixup`则和`squash`不同，它会自动忽略此次日志的内容。这正是我此次commit整理中需要的

如上，我就会得到一个这样的日志：

{% highlight console %}
[root@addcp gitdemo]# git log
commit 3e5d79d149942f7dc0741a9a42ce43d98beafcd3
Author: admin <admin@addcp.com>
Date:   Sat Sep 7 01:28:51 2013 +0800

    update test.txt

commit 0e80061e90503189dd7f189297aed809953e5e8b
Author: admin <admin@addcp.com>
Date:   Sat Sep 7 01:28:22 2013 +0800

    add test.txt
{% endhighlight %}

这样整个世界就都清净了
