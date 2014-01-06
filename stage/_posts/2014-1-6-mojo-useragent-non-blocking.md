---
layout: post
title: Mojo::UserAgent [Non-blocking requests in progress]
category : Perl
---
最近在一个[mojo](http://mojolicio.us/)项目里，其中一个接口会去调用Mojo::UserAgent，按照官方的文档，我把这个接口做成了异步调用。

{% highlight perl %}
get '/title' => sub {
    my $self = shift;

    # 阻塞部分
    my $result = $self->ua->get('example.com');

    # 异步部分
    $self->ua->get('mojolicio.us' => sub {
        my ($ua, $tx) = @_;
        $self->render(data => $tx->res->dom->at('title')->text);
    });
};
{% endhighlight %}

在使用中却发现，一但请求比较密集，就会报`Non-blocking requests in progress`的错。

google了一下发现[Jan Henning Thorsen](https://github.com/jhthorsen)也发现了同样的错误。

这个由`Mojo::UserAgent`报的错，是由于在一个Mojo应用里同时使用了`$self->ua / $c->ua`的`阻塞`和`异步`两种模式。而解决方法也很简单，就是自己用`Mojo::UserAgent->new`再构造一个User::Agent对象给非阻塞部分使用。[原文](https://github.com/marcusramberg/Mojolicious-Plugin-OAuth2/pull/8)

代码大概像这样:

{% highlight perl %}
get '/title' => sub {
    my $self = shift;

    # 阻塞部分改
    my $ua = Mojo::UserAgent->new;
    my $result = $ua->get('example.com');

    # 异步部分
    $self->ua->get('mojolicio.us' => sub {
        my ($ua, $tx) = @_;
        $self->render(data => $tx->res->dom->at('title')->text);
    });
};
{% endhighlight %}
