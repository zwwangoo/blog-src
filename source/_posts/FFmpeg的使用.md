---
title: FFmpeg的常用参数和使用示例
date: 2021-03-26
tags: [FFmpeg, 视频]
---


- [阮一峰——FFmpeg视频处理入门教程](https://www.ruanyifeng.com/blog/2020/01/ffmpeg.html)

FFmpeg 是一个开放源代码的自由软件，可以运行音频和视频多种格式的录影、转换、流功能。这里结合自己的实际使用整理FFmpeg常用的命令和参数含义。

## 安装和使用格式

基于Ubuntu系统，只需要简单的apt命令就可以安装。

```bash
sudo apt update
sudo apt -y install ffmpeg
```

安装完成之后就可以使用FFmpeg，输入`ffmpeg` 就可以查看其版本和配置信息。ffmpeg有许多命令参数，可以分为五个部分：

```
ffmpeg {1} {2} -i {3} {4} {5}
```

- 1.全局参数，例如`-y`,`-v info`等等
- 2.输入文件参数，例如`-c:v libx264`等等
- 3.输入文件
- 4.输出文件参数
- 5.输出文件

使用示例：

```
ffmpeg -y -v info -c:v libx264 -i input.mp4 -c:v libvpx-vp9 -c:a libvorbi output.webm 
```

## 常用命令行参数

常用的命令行参数说明：

- `-y` 不经过确认，输出时直接覆盖同名文件
- `-v info` 指定日志级别，常用的有`info`、`error`
- `-i` 指定输入文件或流地址
- `-c` 指定编码器，一般常用`-c copy` 表示直接复制不进行重新编码
- `-c:v` 指定视频编码器
- `-c:a` 指定音频解码器
- `-an` 去除音频流
- `-vn` 去除视频流
- `-f` 强制使用格式输出，常用有`-f mp4`、`-f flv`、`-f segment` 
- `-r` 指定帧率，缺省25。例如`-r 15`

## 使用示例

### 1、查看文件或流信息

查看视频文件的元信息，比如编码格式和比特率或视频文件的持续时间和分辨率

```
ffmpeg -i input.mp4
```

### 2、视频流转成本地视频文件

将直播流`rtsp`或`rtmp`录制成视频文件

<!-- more -->

```
ffmpeg -y -i rtmp://ip:port/stream -f mp4 out.mp4
```

- `-f` 可以指定参数为 `mp4`、`flv`等
- 如果流格式为`rtsp` 可以使用参数`-f rstp` 指定输入流格式
- 可以使用`-rtsp_transport tcp` 指定 `rtsp` 使用`tcp`协议

```bash
ffmpeg \
-y \
-v info \  # 指定日志级别为info
-rtsp_transport tcp \  # rtsp 使用tcp协议
-i "rtsp://ip:port/stream" \
-f mp4 \ # 指定视频文件格式为mp4
out.mp4
```

### 3、从视频文件中截取一帧

截取视频文件第一帧输出为`jpg`文件：

```bash
ffmpeg -y -i input.mp4 -vframes 1 -f mjpeg output.jpg
```

- `-vframes 1` 指定帧数，这里指定1帧
- `-f` 输出图片格式，`-f mjpeg` 指定为`jpg` ; `-f image2` 指定输出格式为`png`

如果想从指定时间截取需要使用`-ss` 参数指定开始时间

```bash
ffmpeg -y -ss 00:10:00 -i input.mp4 -vframes 1 -f mjpeg output.jpg
```

- `-ss` 指定开始时间，格式可以是：`hh:mm:ss.xxx`，也可以是秒

如果需要持续截取多张图片，可以使用以下命令：

```bash
ffmpeg -y \
-ss 00:00:10 \  # 指定从视频的第10秒开始截取
-i input.mp4 \
-vframes 10 \  # 只取10帧
-t 10 \  # 持续10秒
-f image2 \  # 指定输出格式为png
output_%3d.png  # 文件格式为output_001.png
```

**注意参数 `-ss` 的位置**

 **在截取本地视频文件时，参数`-ss` 应该在输入`-i`参数之前**。实际过程中多次将 `-ss` 放在`-i` 之后发现随着偏移时间的增大截取一帧的耗时越长，但是将`-ss`放在`-i`之前，就会发现截图时间是恒定和较低的。原因是前者每次都从视频文件的开始时间进行偏移，而后者直接定位当指定时间位置不用处理`-ss`之前的部分。

### 4、视频文件裁剪

### 5、Mp4转Mp3

### 6、合并多个视频文件

多个视频文件需要将视频文件名按照先后顺序保存到`input_list.txt`文件中，例如

```
a.mp4
b.mp4
c.mp4
```

然后通过 `-f concat` 命令合并`input_list.txt`文件中的视频为`output.mp4`

```
ffmpeg  \
-y -v info  \
-f concat -safe 0  \
-i input_list.txt  \
-c copy  \
-bsf:a aac_adtstoasc -movflags +faststart  \
output.mp4

```

这种方式是成功率很高，也是最好的

### 7、流文件处理

实际业务中处理过录制直播流按照指定时长切分成多个视频文件。

```
ffmpeg  \
-y  \
-v info  \
-rtbufsize 1m \
-i rtmp://host:port/stream \
-movflags faststart+frag_keyframe  \      # 使mp4支持渐进式下载
-c:v copy  \                              # 原始编解码数据必须被拷贝
-c:a copy \                               # 设定声音编码，降低CPU使用
-f segment \                              # 输出流切片
-segment_format mp4 \                     # 流输出格式
-strftime 1  \                            # 设置切片名为生成切片的时间点
-segment_time 30   \                      # 流切分时长30秒
-reset_timestamps 1  \                    # 每个切片都重新初始化时间戳
out_file__%Y-%m-%d_%H-%M-%S.mp4           # 输出文件名
```

### 8、输出到管道

