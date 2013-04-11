---
layout: post
title: 在Jekyll中使用截断输出
category : Jekyll
tagline: "Supporting tagline"
tags : [excerpt, jekyll, github]
---

因为之前一直用的是[WordPress](http://wordpress.org)，转到Jekyll后自然的就想去找一个可以识别`<!-- more -->`来截断输出的方法

### 插件

Goolge了一下，果然很快就找到相关插件：

-   [`jekyll-only_first_p`](https://github.com/sebcioz/jekyll-only_first_p)：可以只输出第一段的插件(*需要[Nokogiri](http://nokogiri.org/)支持*)
-   [`excerpt.rb`](https://gist.github.com/stympy/986665)：可以识别`<!-- more -->`的插件

不过如果你的Jekyll也是托管在Github上的话，那么就不能用插件的方法了。
出于安全考虑，Github在运行Jekyll的时候用了`--safe`的参数，第三方插件通通无效。
### Liquid

正当我有点小失望的时候，找到了这篇文章：[Post excerpts in Jekyll](http://foldl.me/2012/jekyll-excerpts/)
<!-- more -->

只要利用Liquid模板语言中的一个filter就可以实现：

{% capture text%}{%raw%}{{ post.content | split: '<!-- more -->' | first }}{%endraw%}{% endcapture %}
{% include liquid_raw%}

然后在截断的文章后加上`阅读全文`

    <p><a href="{{ post.url }}#more">阅读全文 →</a></p>

这样，即使是托管在Github上的Jekyll也能截断输出了。
