---
title: AMD RX 6700XT 原生 ROCm7 驱动 ComfyUI 攻略
date: 2026-02-18 12:46:35 +0800
categories: [计算机技术,AI工具]
tags: [ROCm,ComfyUI,AMD显卡,PyTorch]
---

ROCm7的推出让AMD显卡可原生驱动ComfyUI，无需再依赖ZLUDA转译，7000系以上显卡可直接用官方版，而RX 6700XT这类6000系显卡也能通过手动配置实现原生运行，本文就为大家详细梳理RX 6700XT的完整安装、配置与启动步骤，亲测有效。

## 适用显卡范围

本攻略适配基于gfx103x架构的AMD显卡，除RX 6700XT外，还支持以下型号：

- gfx1030：AMD RX 6800 / XT
- gfx1031：AMD RX 6700 / XT
- gfx1032：AMD RX 6600
- gfx1033：AMD Van Gogh iGPU
- gfx1034：AMD RX 6500 XT
- gfx1035：AMD Radeon 680M Laptop iGPU
- gfx1036：AMD Raphael iGPU

## 前期准备

1. **系统要求**：Windows 10/11 64位系统（建议Win11 22H2及以上版本）
2. **工具优势**：本次使用**UV**创建虚拟环境，相比传统venv可自动下载匹配的Python版本，比Conda更轻量便捷；需提前确保电脑联网，且关闭各类杀毒软件的实时防护（避免安装包被误删）。

## 步骤1：安装/更新AMD官方驱动与HIP SDK

### 1.1 安装最新AMD显卡驱动

前往AMD官方驱动下载页：[https://www.amd.com/zh-cn/support/download/drivers.html](https://www.amd.com/zh-cn/support/download/drivers.html)，选择自己的显卡型号（RX 6700XT）和对应系统，下载并安装最新版WHQL驱动，安装完成后重启电脑。

> 若此前安装过旧版AMD驱动，建议使用AMD官方清理工具 `amd--cleanup--utility.exe`彻底清除残留，避免版本冲突。

### 1.2 安装最新HIP SDK

HIP SDK是ROCm平台的核心组件，前往官方下载页：[https://www.amd.com/zh-cn/developer/resources/rocm-hub/hip-sdk.html](https://www.amd.com/zh-cn/developer/resources/rocm-hub/hip-sdk.html)，选择适配Windows 10/11的ROCm版本（建议6.4.2及以上），下载并默认安装即可。

## 步骤2：下载ComfyUI本体并预处理配置

### 2.1 克隆ComfyUI仓库

打开电脑的命令提示符（CMD）或PowerShell，执行以下命令克隆ComfyUI官方仓库到本地（可自定义存放路径）：

```bash
git clone https://github.com/Comfy-Org/ComfyUI.git
```

克隆完成后，进入ComfyUI根目录：

```bash
cd ComfyUI
```

### 2.2 预处理requirements.txt文件

用记事本或任意代码编辑器打开ComfyUI根目录下的 `requirements.txt`，**注释掉**以下4行内容（避免后续安装时覆盖自定义的ROCm版PyTorch）：

```
# numpy>=1.25.0
# torch
# torchvision
# torchaudio
```

保存并关闭文件即可。

## 步骤3：下载ROCm7专属SDK与PyTorch安装包

由于6000系显卡无官方原生ROCm7安装包，需下载适配gfx103x架构的特殊版本安装包（共7个，包含ROCm SDK和PyTorch相关whl包），下载地址：[https://drive.google.com/drive/folders/1ULSlS5-GMqiwW_1jDBIHDsgUkwac97Y6?usp=drive_link](https://drive.google.com/drive/folders/1ULSlS5-GMqiwW_1jDBIHDsgUkwac97Y6?usp=drive_link)

> 注意：下载后若出现安装失败，大概率是安装包损坏，需重新下载；下载完成后，将所有7个安装包**放到ComfyUI根目录下**，方便后续安装。

## 步骤4：配置Python3.12虚拟环境并安装相关包

本次特殊版PyTorch仅支持Python3.12，而若你的电脑是Python3.13，无需卸载，通过UV创建Python3.12虚拟环境即可，全程在ComfyUI根目录的CMD/PowerShell中执行命令。

### 4.1 安装UV工具

```bash
pip install uv
```

### 4.2 创建并激活Python3.12虚拟环境

UV会自动下载Python3.12，无需手动安装，执行命令：

```bash
uv venv --python 3.12
```

创建完成后，激活虚拟环境：

```bash
.venv\Scripts\activate
```

激活成功后，命令行前缀会出现 `(.venv)`标识。

### 4.3 分两步安装ROCm SDK与PyTorch包

由于安装包之间存在依赖关系，需分两次安装，**直接执行以下命令即可**（确保安装包均在ComfyUI根目录）：

#### 第一步：安装ROCm7.1.1 SDK相关包

```bash
uv pip install "rocm-7.1.1.tar.gz" "rocm_sdk_libraries_gfx103x_all-7.1.1-py3-none-win_amd64.whl" "rocm_sdk_devel-7.1.1-py3-none-win_amd64.whl" "rocm_sdk_core-7.1.1-py3-none-win_amd64.whl"
```

#### 第二步：安装ROCm版PyTorch相关包

```bash
uv pip install "torch-2.9.1+rocmsdk20251207-cp312-cp312-win_amd64.whl" "torchaudio-2.9.0+rocmsdk20251207-cp312-cp312-win_amd64.whl" "torchvision-0.24.0+rocmsdk20251207-cp312-cp312-win_amd64.whl"
```

### 4.4 安装ComfyUI基础依赖与管理器依赖

```bash
# 安装ComfyUI核心依赖
uv pip install -r requirements.txt
# 安装ComfyUI-Manager管理器依赖（方便后续安装自定义节点）
uv pip install -r manager_requirements.txt
```

## 步骤5：导入模型并启动ComfyUI

### 5.1 导入Stable Diffusion模型

将你下载的Stable Diffusion模型（ckpt/safetensors格式）放到ComfyUI根目录的 `models/checkpoints`文件夹下；若有VAE、LoRA等模型，分别放到对应 `models/vae`、`models/loras`文件夹即可。

### 5.2 启动ComfyUI

确保虚拟环境处于激活状态（`(.venv)`标识存在），在ComfyUI根目录执行启动命令：

```bash
python.exe main.py --windows-standalone-build
```

### 5.3 访问ComfyUI界面

启动成功后，CMD/PowerShell会出现提示：`To see the GUI go to: http://127.0.0.1:8188`，打开任意浏览器，输入该地址即可进入ComfyUI的可视化操作界面，至此RX 6700XT原生ROCm7驱动ComfyUI完成！

## 后续启动方式

每次重启电脑后，无需重新配置环境，仅需执行以下步骤即可启动：

1. 打开CMD/PowerShell，进入ComfyUI根目录：`cd ComfyUI`
2. 激活虚拟环境：`.venv\Scripts\activate`
3. 启动ComfyUI并启动manager插件：`python.exe main.py --windows-standalone-build --enable-manager`
4. 浏览器访问 `http://127.0.0.1:8188`即可。

## 常见问题解决

1. **安装包损坏**：下载后安装失败，重新下载所有7个安装包，确保下载过程中网络稳定，关闭杀毒软件。
2. **虚拟环境激活失败**：检查命令是否输入正确，确保在ComfyUI根目录执行，且UV已成功安装。
3. **启动后显卡未识别**：确认AMD驱动和HIP SDK为最新版，虚拟环境已激活，且安装包为gfx103x架构专属版本。
4. **模型加载失败**：确认模型格式为ckpt/safetensors，且放到对应 `models`子文件夹下，无路径中文。

## 参考资料

[AMD 780M +ROCm7+pytorch+ComfyUI_windows_portable_amd实现在windows下原生ai绘图](https://blog.csdn.net/hao17119/article/details/157844328)