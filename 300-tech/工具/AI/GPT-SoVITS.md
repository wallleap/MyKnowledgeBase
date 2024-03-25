---
title: GPT-SoVITS
date: 2024-03-21 13:53
updated: 2024-03-21 14:39
---

TTS 模型、语音克隆

## 安装预先条件

- Python 3.9 以上
- conda

[Miniconda](https://docs.anaconda.com/free/miniconda/) 安装：

```sh
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh
~/miniconda3/bin/conda init bash # 或 ~/miniconda3/bin/conda init zsh
```

接着把这个目录写到 `.bashrc` 中

```sh
vim ~/.bashrc
```

添加这条

```sh
export PATH=~/miniconda3/bin:$PATH
```

source 一下

```sh
source ~/.bashrc
```

## 开始安装

项目地址：<https://github.com/RVC-Boss/GPT-SoVITS>

```sh
conda create -n GPTSoVits python=3.9
conda activate GPTSoVits
git clone https://github.com/RVC-Boss/GPT-SoVITS.git
bash install.sh
# 一路回车
```
