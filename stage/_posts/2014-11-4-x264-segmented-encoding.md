---
layout: post
title: x264 分段编码(segmented encoding)防花屏的选项
category: encoding
tags : [x264]
---
最近使用x264的2pass编码对同一个文件进行分段编码，然后再合并，实现多进程转码进而可以进行分布式集群转码从而加速转码速度。

{% highlight bash %}
$ mencoder -really-quiet /root/video/2159034.mp4 -ss 0 -endpos 60 -of rawvideo -ovc raw -nosound  -ofps 23  -vf-add scale=404:720 -vf-add format=i420 -vf-add hqdn3d -vf-add harddup -o - 2>/dev/null |x264 --no-progress --preset slower --tune film --weightp 2 --keyint 250 --min-keyint 25 --non-deterministic --bframes 3 --no-fast-pskip --qcomp 0.6 --merange 24 --threads auto --input-csp i420  --level 3.1 --profile high --crf 20 --input-res 404x720 --fps 23 - --pass 1 --stats /tmp/video/1415072454_part1/3000/part_12159034/1415072454.passLog -o /dev/null

$ mencoder -really-quiet /root/video/2159034.mp4 -ss 0 -endpos 60 -of rawvideo -ovc raw -nosound  -ofps 23  -vf-add scale=404:720 -vf-add format=i420 -vf-add hqdn3d -vf-add harddup -o - 2>/dev/null |x264 --no-progress --preset slower --tune film --weightp 2 --keyint 250 --min-keyint 25 --non-deterministic --bframes 3 --no-fast-pskip --qcomp 0.6 --merange 24 --threads auto --input-csp i420  --level 3.1 --profile high -B 3000 --input-res 404x720 --fps 23 - --pass 2 --stats /tmp/video/1415072454_part1/3000/part_12159034/1415072454.passLog -o part1.264
{% endhighlight %}

但是在转码过程中遇到了问题，单独的编码输出的`.264`文件再加上mp4的封装都可以正常播放。但是将`.264`文件合并为一个文件之后，快进时播放就会花屏。

{% highlight bash %}
$ MP4Box -add part1.264#video -cat part2.264 -fps 29.97 -add video.m4a#audio -new result.mp4
{% endhighlight %}

在重新转码多次之后依然得到相同的结果，说明单纯的append file是导花屏的原因。

在Google之后终于发现是由于x264默认会对输出的`.264`文件头部进行优化，导致合并`.264`文件的时候出现错误。只要简单的加上`--stitchable`这个选项就可以避免花屏了。

    --stitchable        Don't optimize headers based on video content
                        Ensures ability to recombine a segmented encode

结果如下：

{% highlight bash %}
$ mencoder -really-quiet /root/video/2159034.mp4 -ss 0 -endpos 60 -of rawvideo -ovc raw -nosound  -ofps 23  -vf-add scale=404:720 -vf-add format=i420 -vf-add hqdn3d -vf-add harddup -o - 2>/dev/null |x264 --no-progress --stitchable --preset slower --tune film --weightp 2 --keyint 250 --min-keyint 25 --non-deterministic --bframes 3 --no-fast-pskip --qcomp 0.6 --merange 24 --threads auto --input-csp i420  --level 3.1 --profile high --crf 20 --input-res 404x720 --fps 23 - --pass 1 --stats /tmp/video/1415072454_part1/3000/part_12159034/1415072454.passLog -o /dev/null

$ mencoder -really-quiet /root/video/2159034.mp4 -ss 0 -endpos 60 -of rawvideo -ovc raw -nosound  -ofps 23  -vf-add scale=404:720 -vf-add format=i420 -vf-add hqdn3d -vf-add harddup -o - 2>/dev/null |x264 --no-progress --stitchable --preset slower --tune film --weightp 2 --keyint 250 --min-keyint 25 --non-deterministic --bframes 3 --no-fast-pskip --qcomp 0.6 --merange 24 --threads auto --input-csp i420  --level 3.1 --profile high -B 3000 --input-res 404x720 --fps 23 - --pass 2 --stats /tmp/video/1415072454_part1/3000/part_12159034/1415072454.passLog -o part1.264
{% endhighlight %}
