----
title: ffmpeg 视频压缩
date: 2018-10-14 19:47:00
tags: ffmpeg
----

# 目标大小模式
``` bash
ffmpeg -i /path/to/input -fs SIZE /path/to/output
```

例如：

``` bash
ffmpeg -i funny.mov -fs 300MB funny.mp4
```
