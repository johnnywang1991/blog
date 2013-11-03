---
layout: post
title: 爬虫的url去重
category : Perl
---
最近用Perl写了一些爬虫，没想到在url去重的问题上碰了钉子。

之前写过一些小的爬虫，url去重的方法都是爬一条就写进数据库，然后在写之前去查询数据库。
因为这次爬虫的规模较之前的都大，如果在去重问题上消耗太多时间和数据库连接实在不值得。

在网上一搜，才发现url去重也是爬虫的难题之一。而我最终采用了[不简单的URL去重](http://blog.csdn.net/historyasamirror/article/details/6746217)这篇文章中推荐的方法：**使用BerkeleyDB**。

### 什么是BerkeleyDB

[`维基百科 BerkeleyDB`](http://zh.wikipedia.org/wiki/Berkeley_DB)：

> Berkeley DB（BDB）是一个高效的嵌入式数据库编程库，C语言、C++、Java、Perl、Python、Tcl以及其他很多语言都有其对应的API。Berkeley DB可以保存任意类型的键/值对（Key/Value Pair），而且可以为一个键保存多个数据。

> Berkeley DB支持让数千的并发线程同时操作数据库，支持最大256TB的数据，广泛用于各种操作系统，其中包括大多数类Unix操作系统、Windows操作系统以及实时操作系统。

<!-- more -->

### 使用BerkeleyDB对url去重的好处

1. BerkeleyDB是一种放在磁盘上的url解决方案，相对于将url全部放进hash这种方法，不用担心内存溢出的问题，在程序意外中断后也不会丢失数据。

2. 虽然是放在磁盘上的解决方案，但不需要考虑进行磁盘IO操作的性能损失的，BerkeleyDB在设计的时候很好地考虑了这些问题。

3. 相对于[Bloom Filter](http://en.wikipedia.org/wiki/Bloom_filter)，BerkeleyDB不存在失误率的问题。

4. BerkeleyDB是一个key/value对的数据库，这也是使用它作为url去重的原因。

5. BerkeleyDB还是一种嵌入型数据库，它是和程序跑在同一内存空间内的。其结果是，不管应用程序是运行在同一台机器上还是运行在网络上，在进行数据库操作时，它都无需进行进程间通信。相对于一般C/S的数据库，效率高了很多。

6. BerkeleyDB是一种NOSQL数据库，不必在解析SQL语句上消耗不必要的资源。

### 例子

BerkeleyDB有Perl的接口，用法也非常简单，下面贴一段我的部分代码：

{% highlight pl %}
use 5.010;
use strict;
use warnings;
use Smart::Comments;
use utf8;

use BerkeleyDB;
use Mojo::UserAgent;

# 使用Mojo::UserAgent用于抓取网页
my $ua = Mojo::UserAgent->new;

# temp file for BerkeleyDB
my $tmp_dir = "/tmp";
my $berkeleydb_temp_file = "/tmp/tmp.berkeleydb";

# 设定BerkeleyDB的环境
my $env = new BerkeleyDB::Env
    -Home   => $tmp_dir,
    -Flags  => DB_CREATE|DB_INIT_CDB|DB_INIT_MPOOL
        or die "can not open environment: $BerkeleyDB::Error\n";

# 将%data与BerkeleyDB绑定
my $db = tie my %data, 'BerkeleyDB::Hash',
    -Filename   => $berkeleydb_temp_file,
    -Flags      => DB_CREATE,
    -Env        => $env
        or die "Can not create file: $! $BerkeleyDB::Error\n";

# usage: $db->db_put('apple', 'red');

# 在页面上获取一些url
# do some thing ...
# ...
# ...

# 从页面上获得一些url(作为例子)
my @url = qw/a.example.com b.example a.example/;

# 使用Mojo::IOLoop作异步抓取页面
# 详细用法见MetaCPAN的Mojo::IOLoop用法
Mojo::IOLoop->delay(
    sub {
        my $delay = shift;
        for (@url) {
            # url 去重
            next if $data{$_};
            # 抓取网页内容
            $ua->get($_ => $delay->begin);
        }
    },
    sub {
        my ($delay, @result) = @_;

        for my $tx (@result) {
            if (my $dom = $tx->success) {

                # 使用lock保证多进程时BerkeleyDB的数据安全
                my $lock = $db->cds_lock();

                # save to berkeleydb
                $db->db_put($url, 'done');

                $db->db_sync();
                $lock->cds_unlock();
                undef $lock;

                # do something...
            } else {
                my ($err, $code) = $tx->error;
                say $code ? "$code response: $err" : "Connection error: $err";
            }
        }
    }
);

Mojo::IOLoop->start unless Mojo::IOLoop->is_running;
{% endhighlight %}
