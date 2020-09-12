---
title: 使用youtube-dl下载YouTube播放列表
date: 2020-09-12 10:22:32
tags:
- YouTube 
- FFmpeg
categories:
- 音视频
---
本人是一个重度的音频依赖者，有很多国内的站点因为版权原因不能放松的视频在YouTube上都能找到，但是我又特别喜欢听某些剧集的音频，以前一些比较少的剧集我都是通过Chrome的插件获取到音频地址从而保存为本地的webm然后通过FFmpeg转换为MP3格式（因为要在车上听）。

现在遇到一个特别喜欢的剧集但是有120多集实在无法人肉通过手动点击的办法一集一集的下载，所以我找到了一个堪称下载神器的[youtube-dl](https://github.com/ytdl-org/youtube-dl)工具，youtube-dl参数非常的详细可以支持相当多的下载粒度。值得每个视频搬运工详细的研究。这里我只说一下批量下载剧集的办法。

### 1.首先获取剧集的url
```
它应该是一个这样的url
https://www.youtube.com/user/TheLinuxFoundation/playlists
```

### 2.命令格式
```
 youtube-dl -x --audio-format mp3 -o '%(title)s.%(ext)s' https://www.youtube.com/user/TheLinuxFoundation/playlists

```
#### *2.1* --audio-format mp3 保存为MP3格式，具体的流程是先下载一个临时的m4a原文件，然后调用FFmpeg转存为MP3文件。所以一定注意在使用youtube-dl时必须安装了ffmpeg。

#### *2.2* -o '%(title)s.%(ext)s' [详细参阅](https://github.com/ytdl-org/youtube-dl#output-template) 输出格式，如果使用默认参数会出现以剧集名+视频ID的名字为文件名，比如  LINUX教程09-dsdhCGS.mp3  这种命名格式，对于分集比价多的播放列表不利于后期整理。

我现在挂在google cloud的虚拟机上下载，下载速度能跑到15M左右.