+++
date = '2025-09-26T15:27:05+08:00'
draft = false
title = 'Mineru'
tags = ['python']
+++

# 用MinerU将PDF转成epub

看到一本电子书是扫描版的PDF格式，文件很大而且阅读起来不方便。所以尝试用MinerU将PDF转换成epub。整体流程如下：

1. 将PDF转换成Markdown格式
2. 将Markdown转换成epub

# MinerU
MinerU是一个处理PDF的工具 地址[github](https://opendatalab.github.io/MinerU/) 。MinerU是一款将PDF转化为机器可读格式的工具（如markdown、json），可以很方便地抽取为任意格式。

## 部署
在win平台，可以用docker一键部署。如果遇到网络问题，可以在Dockerfile中将配置huggingface改为使用modelscope 

```
docker run --gpus all --shm-size 32g -p 30000:30000 -p 7860:7860 -p 8000:8000 --ipc=host -it mineru-vllm:latest
```

## 使用
配置好环境后 MinerU提供了多种调用方式：

- 命令行：`mineru -p abc.pdf -o output --source local`
- python脚本。
- API （测试中）

因为要批量执行，所以我用python脚本完成，参考我的项目 [pdf2epub](https://github.com/didaman/pdf2epub)


## 限制虚拟显存
解决方案
如果显卡显存超过8G，遇到大文件时会启动batch模式。但是又会爆显存，出现程序被强行杀掉的情况。
```
Killed  
```

因此需要限制虚拟显存
```bash
export MINERU_VIRTUAL_VRAM_SIZE=6
```

同时可以将大PDF拆分成多个小的PDF文件。逐个调用脚本。具体参考python脚本

# 把Markdown转成epub

借助python的markdown和ebooklib库，可以很容易地实现。