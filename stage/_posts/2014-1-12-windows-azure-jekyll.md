---
layout: post
title: 在Windows Azure上搭建Jekyll
category: jekyll
---
刚刚收到Windows Azure发来的邮件

> 诚邀您参加由世纪互联运营的 Windows Azure beta 免费试用 

我很幸运的得到了Windows Azure beta的试用VPS，而且也没有说多久过期，看来起码可以免费用上三个月以上了。

当然这个VPS是在国内的网络里，`搭梯子`什么的当然做不到。但是还是可以搭个博客什么的。

于是我把在github Pages上跑的Jekyll博客也移了过来（起码写博客不用开虚拟机，直接登到VPS上就可以了）。

### 搭建一个Jekyll环境。

在Windows Azure创建虚拟机的时候我选了CentOS（主要是工作中也在用），然后下一步，下一步。。这样一个虚拟机就创建完毕。

然后打开一个ssh终端，输入在刚才创建虚拟机时候填的主机名、端口、用户名、密码，然后登录到虚拟机上。

登入之后第一件事第一件事就是编辑`/etc/yum.conf`文件：

{% highlight bash %}
$ vim /etc/yum.conf
{% endhighlight %}

{% highlight vim %}
[main]                                                                                                                                     
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=0
debuglevel=2
logfile=/var/log/yum.log
exactarch=1
obsoletes=1
gpgcheck=1
plugins=1
installonly_limit=5
bugtracker_url=http://bugs.centos.org/set_project.php?project_id=16&ref=http://bugs.centos.org/bug_report_page.php?category=yum
distroverpkg=centos-release
# exclude=kernel*
#  This is the default, if you make this bigger yum won't see if the metadata
# is newer on the remote and so you'll "gain" the bandwidth of not having to
# download the new metadata and "pay" for it by yum not having correct
# information.
#  It is esp. important, to have correct metadata, for distributions like
# Fedora which don't keep old packages around. If you don't like this checking
# interupting your command line usage, it's much better to have something
# manually check the metadata once an hour (yum-updatesd will do this).
# metadata_expire=90m

# PUT YOUR REPOS HERE OR IN separate files named file.repo
# in /etc/yum.repos.d
{% endhighlight %}

把其中`exclude=kernel*`注释掉，不然装包的时候会报错的。

由于Jekyll需要Ruby环境，所以在终端执行下面的命令，安装Ruby

{% highlight bash %}
$ curl -L https://get.rvm.io | bash -s stable --ruby
{% endhighlight %}

Jekyll安装则按照Jekyll官方文档上说的，在终端执行：

{% highlight bash %}
$ gem install jekyll
{% endhighlight %}

但是我在执行这条命令时发现，gem迟迟没有反应。再执行：

{% highlight bash %}
$ gem install jekyll -V
{% endhighlight %}

这里发现，没有反应是因为gem的源被国内的网络环境弄的无法连接（你懂的）

果断按照[http://ruby.taobao.org/](http://ruby.taobao.org/)上的文档，删除了现在无法连接的源，更新了淘宝的源。

{% highlight bash %}
$ gem sources --remove https://rubygems.org/
$ gem sources -a http://ruby.taobao.org/
$ gem sources -l
*** CURRENT SOURCES ***

http://ruby.taobao.org
# 请确保只有 ruby.taobao.org
{% endhighlight %}

然后再执行：
{% highlight bash %}
$ gem install jekyll
{% endhighlight %}

果然很快很顺利。^_^

---------
**写在后面**

不过我现在还是不清楚，能不能把没备案的域名解析过来。。。所以我还是在用那个又长又难看又难记的`xxxxxx.chinacloudapp.cn`。。=w=
