+++
date = '2025-05-13T15:38:42+08:00'
draft = false
title = '如何从 SACD 的 ISO 中提取音频'
categories = ['Main Sections']
tags = ['music']
+++

SACD 是 Super Audio CD的缩写，是超级音频光盘系统，是索尼和飞利浦在它们联合开发的 MMCD （单面双层结构的高密度光碟）基础上研制推出的新数字音频格式。音频采样率远大于普通 CD 。

和普通的 CD 不同， SACD 采用差分编码方式，所以需要特定的设备播放；或者转码为绝对编码，在普通设备中播放。本文将介绍如何把 SACD 的 ISO 中的音频提取并转化为绝对编码。

## 第一步：观察
打开 .iso 文件，里面有一些 .TOC 文件和 .2CH 文件。这就说明这个 .iso 文件大概率是来自 SACD 的。

## 第二步
下载 [sacd_extract](https://github.com/sacd-ripper/sacd-ripper/releases)

输入

```shell
.\sacd_extract.exe -i "xxx.iso" -C -o "E:\outputDir"
```

输出 .cue 文件，查看基本信息，以及查看有几个 TRACK 。

输入

```shell
.\sacd_extract.exe -i "xxx.iso" -p -k -t 1,n -o "E:\outputDir"
```

把全部的 TRACK 合并为一个 .dff 格式音频文件， `n` 为刚刚查看的 TRACK 数量， `-p` 是指输出为 .dff 格式， `-s` 是指输出为 .dsf 格式，具体查看软件使用手册。

## 第三步
按需修改 .cue 文件，使用其他脚本，读取 .cue 文件，来对 .dff 文件进行转换格式处理。

比如 [SplitAlbums](https://github.com/tsctsc6/SplitAlbums)

## 第四步
使用 ffmpeg 检查音频参数：

```shell
ffprobe -v error -show_entries stream=sample_rate,sample_fmt,bits_per_sample,bits_per_raw_sample yourfile.flac
```