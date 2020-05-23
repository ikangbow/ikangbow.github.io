---
title: ffmpeg的使用
date: 2020-04-04 08:22:08
category: java
tags: java
---

## 视频格式小科普

在开始下面的教程之前有必要先简单科普一下视频格式的知识。

视频格式是一种非常不专业的叫法，事实上，视频有编码格式和容器格式两种。编码格式之于容器格式就像牛奶之于杯子一样。 常见的视频文件有mp4(mpeg4 part 14)，mkv，flv等，这些是视频的容器格式/封装格式(Container format)。它们包含视频流和音频流，mkv支持多条音轨和字幕，因此是目前最受欢迎的容器格式。比如在播放mkv视频的时候，可以选择不同语言的音轨/字幕。 至于视频编码格式(Encoding format)，则是这些容器所包含的视频流/音频流所采用的压缩方式，比如最常用的h.264/aac，mpeg4/mpeg3等。

### 安装ffmpeg

在mac os上安装ffmpeg：

brew install ffmpeg --with-fdk-aac --with-libass --with-sdl2
与默认安装相比，这里增加了对fdk-aac音频库和ass字幕库的支持，同时安装了ffplay播放器。

在linux上安装ffmpeg最简单的方式是使用官方提供的静态编译安装包，解压后即可使用。

ffmpeg命令通用格式
先来认识一下ffmpeg命令行工具的格式：

`ffmpeg -global_options -input_1_options -i input_1 -input_2_options -i input_2 -output_1_options output_1` ...
不管你看到的ffmpeg命令多么复杂，万变不离其宗，都可以按照这个格式拆分成几个单独的部分，各个选项的含义可以查阅ffmpeg的documentation。

需要注意的是： * 输入输出 * 至少有一个输入和输出 * 可以同时有多个输入和输出 * 输入、输出不一定是文件，可以是rtmp流的地址、摄像头对应的设备文件等 * 顺序很重要 * 所有的输入完了然后是输出 * 某个选项只对它后面的输入或输出起作用

### 1、查看视频的信息

`ffmpeg -hide_banner -i input.mkv  或者  ffprobe -hide_banner input.mkv`


通过ffmpeg可以查看一个视频文件的以下基本信息：

视频/音频的编码格式
分辨率(1080p or 1080i, Progressive Interlaced)
时长(duration)/码率(bitrate)/帧率(fps,frames per second)
音轨(Audio)/字幕(Subtitle)

### 2、转码(transcode)

转码是指对流(视频、音频和字幕)进行解码然后再编码，这个过程非常耗CPU。输出的编码格式通过-codec指定，如果不指定，ffmpeg并不会直接copy，而是采用根据容器格式采用相应的默认编码格式进行重新编码。

为了简单起见，把转换容器格式也归为转码，ffmpeg会通过输出文件的后缀判断输出的容器格式，因此不需要指定(-f)。

视频
将mkv转换成mp4

有时候我们需要将mkv转换成mp4：

很多视频网站不支持上传mkv格式
大部分视频剪辑软件，比如premiere等不支持导入mkv格式的视频
下面的命令将mkv的视频和音频流重新封装成mp4文件，不进行重新编码(无损)

`ffmpeg -i input.mkv -codec copy output.mp4`
制作mkv

mkv格式能够封装多个音频、字幕轨，是目前最为流行的视频分发格式，通常使用mkvtoolnix工具来编辑制作mkv视频，通过ffmpeg也能完成一些简单的任务。

将mpeg4/mp3编码的视频重新编码为h.264/aac，并和字幕一起封装成mkv文件：

`ffmpeg -i input.avi -i input.srt -map 0:0 -map 0:1 -map 1:0 -c:v libx264 -c:a aac -c:s srt output.mkv`
这里使用ffmpeg自带的aac音频编码器，而不是fdkaac。

音频
ffmpeg不仅能处理视频，音频文件也不在话下，因此，同样可以用ffmpeg转换音频格式。

将flac无损的音乐转换成高质量(High Quality，即320Kb)的mp3格式：

`ffmpeg -i "Michael Jackson - Billie Jean.flac" -ab 320k "Michael Jackson - Billie Jean.mp3"`

### 3、截取(cut)

从视频中截取出精彩片段是我们最常用的功能。

从第2分钟开始，截取30秒。所有流都直接拷贝。

`ffmpeg -ss 00:02:00.0 -i input.mkv -t 30 -c copy output.mkv`
-ss设置开始时间点，格式是HH:MM:SS.xxx或sec.msec。当作为输入选项时，可以快速seek到指定时间点； -t作为输出选项，设置时长。

截取最大的问题是难以做到精确控制时间点，大部分的视频剪辑软件都存在这样的问题：

视频开头卡顿/不流畅，画面与声音不同步
时间误差甚至能达到秒级，比如截取10秒的片段，可能会得到12秒的输出
原因是ffmpeg会seek到指定时间点前的一个关键帧，如果是copy视频流，seek点和起点之间的额外部分将会被保留，因此就产生了误差。解决办法是将-ss作为输出选项或进行重新编码：

`ffmpeg -i input.mkv -ss 00:02:00.0 -t 30 -c copy output.mkv`
-ss作为输出选项时，解码到指定的时间点，然后开始输出，相当于要扫描前面的所有帧。但是这种方式能获得更精确的输出。

如果还是出现上述问题，使用下面的终极解决方案(使用两次seek)：

`ffmpeg -ss 00:01:30.0 -i input.mkv -ss 00:00:30.0 -t 30 output.mkv`

### 4、截图(screenshot)

截图就是将某个视频帧保存为图片，例如：

在指定时间点截图

`ffmpeg -ss 00:30:14.435 -i input.mkv -vframes 1 out.png`

* -ss作为输入选项，可以快速定位到指定时间点；如果作为输出选项，需要读取指定时间点前面所有的帧，但可以获得更精确的位置。 * -vframes设置输出的帧数。

结合fps视频滤镜，可以从视频中截取多张图片。例如：

每1分钟截一张图，在截图文件名中添加当前的时间

`ffmpeg -i input.mkv -vf fps=1/60 -strftime 1 out_%Y%m%d%H%M%S.jpg`

### 5、压缩(compress)

一张蓝光原盘接近50G，而720p的电影可能只有2G，这是通过压缩实现的。在保证画质的前提下使用更小的文件存储，一直是压缩的目标。然而压缩并非那么简单，比如同样是720p的电影，有的只有2个G，有的却有5个G，然而从视觉上几乎看不出区别来。 这里只介绍压缩的一个简单用途：resize。

将1080p转成720p，宽度自适应

`ffmpeg -i input.mkv -c copy -c:v libx264 -vf scale=-2:720 output.mkv`
-vf是-filter:v的简写，-filter指定滤镜，:v是流选择器，表示对视频流应用滤镜。scale滤镜后面的参数是w:h，w和h可以包含一些变量，比如原始的宽高分别为iw和ih。当其中一个是负数时，假设为-n，ffmpeg自动使用另一个值按照原始的宽高比(aspect ratio)计算出该值，并且保证计算出来的值能被n整除。

因为scale滤镜是针对未编码的原始数据，所以视频流不能用copy，需要进行重新编码。-c copy -c:v libx264表示视频流使用h.264重新编码，其他流直接copy，顺序不能颠倒。

### 6、字幕(subtitle)

对于视频发布者，给视频添加字幕很有必要。如果是mkv，可以采用制作mkv的方法将字幕文件封装到mkv文件里。如果是mp4视频，则需要进行烧录(draw)，即对视频重新编码的过程中将字幕融入视频流。

`ffmpeg -i input.mkv -vf subtitles=input.srt output.mp4`
该过程是通过subtitles滤镜完成的，安装ffmpeg的时候需要启用libass库。subtitles的参数是字幕文件，如果是mkv文件则表示使用mkv的默认字幕流。

如果是ass格式的字幕文件，则使用ass滤镜：

`ffmpeg -i input.mkv -vf ass=input.ass output.mp4`
据我所知，通常ass字幕具有更丰富的样式。

另一种常见的方法是使用-c:s mov_text将字幕流封装进mp4里，它不需要重新编码，但是目前大部分播放器对mp4的软字幕支持不好。

### 7、提取(extract)

有时候我们需要从电影中提取插曲或从视频中提取背景音乐，通过ffmpeg可以很容易从视频文件中提取音频：

`ffmpeg -i cut.mp4 -vn output.mp3`
-vn表示不输出视频流。

输出的mp3默认的码率是128kb，可以使用ab控制。结合截取中的-ss和-t还可以只提取一个部分的音频。

从mkv文件中提取字幕也是同样的方法。

### 8、水印(watermark)

给视频添加水印是通过overlay滤镜实现的，overlay滤镜的作用是将一个视频覆盖另一个视频上面，它有两个输入，前一个是下层(main)，后一个是上层(overlaid)。 overlay的参数x:y是上层左上角的坐标，0:0表示位于下层的左上角。它可以包含以下参数：

W(w):下(上)层的宽
H(h):下(上)层的高
将水印/logo添加到视频的右上角，并且和边缘保持5像素的距离：

`ffmpeg -i input.mkv -i input.png -filter_complex "overlay=W-w-5:5" -c copy -c:v libx264 output.mkv`
input.png是带透明背景的logo，作为上层。

需要注意的是，视频需要重新编码，其他流(音频、字幕)则直接copy，顺序不能颠倒。

### 9、混流(Muxing)

如果我们拍摄了一段视频，想给它添加背景音乐，最好还是借助视频编辑软件，它能对音乐的开头和结尾分别进行增强和减弱的处理，如果不在意这些细节，完全可以用ffmpeg来代替。

替换音频
`ffmpeg -i input.mkv -i input.mp3 -map 0:v -map 1:a -c copy -shortest output.mp4`
-map的作用是手动选择输出的流：

-map 0:v – 从输入0(第1个输入，即input.mkv) 选择视频流
-map 1:a – 从输入1(第2个输入，即input.mp3) 选择音频流
第1个map对应输出的第1个流，第2个map对应输出的第2个流，以此类推。

合并音频
另一个常见的形式是将两个音频合并成一个，需要使用amerge滤镜：

`ffmpeg -i input.mkv -i output.aac -filter_complex "[0:a][1:a]amerge=inputs=2[a]" -map 0:v -map "[a]" -c:v copy -c:a aac -ac 2 -shortest output.mp4`

### 10、制作gif

结合前面讲到的截取和压缩，将输出文件的后缀改为gif就可以得到动态图

`ffmpeg -ss 30 -t 5 -i input.mp4 -r 10 -vf scale=-1:144 -y output.gif`
-r指定帧率，原始的帧率是25，降低帧率能减小gif图片的大小。



但是这种方式的效果很差，有一种粗糙的布料的感觉。 原因在于gif限制只能包含256种颜色，而原始视频可能包含数以百万的颜色，ffmpeg默认会使用一种通用的调色板(palettep)，因此导致颜色失真。2015年，ffmpeg通过引入palettegen和paletteuse两个滤镜来改进这个问题，它通过扫描整个视频，输出一个最佳的调色板，然后再转换成gif的过程中应用这个调色板从而避免颜色失真。

下面的命令从input.mp4的第30秒开始截取5秒，生成高144像素的gif动态图：

`palette="/tmp/palette.png"`
`filters="fps=10,scale=-1:144:flags=lanczos"`

`ffmpeg -ss 30 -t 5 -i input.mp4 -vf "$filters,palettegen" -y $palette`
`ffmpeg -ss 30 -t 5 -i input.mp4 -i $palette -filter_complex "$filters [x]; [x][1:v] paletteuse" -y output.gif`


与前一个输出相比，质量提升了很多。

另一个有待研究的难题是如何控制输出gif的大小。在不影响画质的前提下，我们希望体积越小越好。比如网上有的gif图很清晰，却不超过1M。而一段500K的视频文件，使用上面的方法转成gif后输出的gif却有1.6M。
