---
title: AMD 平台 Stable Diffusion 本地部署速通攻略
date: 2025-9-14 13:29 +0800
categories: [计算机技术,AI工具]
tags: [Stable Diffusion,AI绘画,AMD显卡]
---
（Intel i7-8700K + 24 GB + RX 6700 XT 12 GB 实测通过）

## 零、原理一句话

AMD 卡本身没有 CUDA 核，ZLUDA 把 CUDA 指令流即时转译成 ROCm/HIP，WebUI 把 GPU 当“假 NVIDIA”用，从而跑起 Stable Diffusion。

以下资源均可网络环境免费获取，谨防付费攻略诈骗。目前秋叶大佬在B站有大量高仿号，要求三联获得资源，谨防李鬼！

本文由 Kimi v2 辅助润色。

## 一、准备阶段

1. 全程 SSD，缩短模型加载时间。
2. 显卡驱动 ≥ 23.40（对 ROCm 5.7 兼容性最好）。
3. 保证系统盘 20 GB 以上剩余空间。

## 二、一键包下载与解压

1. 获取秋叶整合包
   链接：[https://www.bilibili.com/opus/897873624905547794](https://www.bilibili.com/opus/897873624905547794) https://pan.quark.cn/s/2c832199b09b
   解压密码：bilibili-秋葉aaaki
2. 解压到任意英文路径，如 `D:\SD-ZLUDA`

## 三、安装 ROCm 5.7

<<<<<<< HEAD
* 首次启动 `A绘世启动器.exe` → 点“一键启动”。
* 弹窗提示缺少 HIP SDK → 弹出官方直链下载 ROCm 5.7 完整版并安装。
* 6700 XT 需手工补丁，其余6800以下显卡，可对应下载必要库：

  - 下载 `ROCmLibs-gfx1031-5.7.7z`
    https://github.com/likelovewant/ROCmLibs-for-gfx1103-AMD780M-APU/releases
  - 关闭所有调用 ROCm 的程序
  - 备份并替换：

    - `rocblas.dll` → `C:\Program Files\AMD\ROCm\5.7\bin\`
    - 整个 `library` 文件夹 → `C:\Program Files\AMD\ROCm\5.7\bin\rocblas\`
* 其他：下面把常见 AMD 显卡型号与 ROCm 里用到的 gfx 代号一一对应，照表找即可。

  （资料来源：ROCm 官方文档与 GitCode 博客，2025-06 更新）

  | 显卡型号                           | 核心代号      | gfx 版本        |
  | ---------------------------------- | ------------- | --------------- |
  | RX 7900 XTX / XT / 7800 XT         | Navi 31       | gfx1100         |
  | RX 7700 XT / 7600 XT               | Navi 32/33    | gfx1102         |
  | RX 6900 XT / 6800 XT / 6800        | Navi 21       | gfx1030         |
  | RX 6700 XT / 6700 / 6600 XT / 6600 | Navi 22/23    | gfx1031         |
  | RX 6500 XT / 6400                  | Navi 24       | gfx1034         |
  | RX 5700 XT / 5700 / 5600 XT / 5600 | Navi 10/12/14 | gfx1010         |
  | RX 5500 XT / 5500                  | Navi 14       | gfx1012         |
  | RX Vega 64 / 56 / VII              | Vega 10/20    | gfx900 / gfx906 |
  | Radeon VII                         | Vega 20       | gfx906          |
  | Instinct MI100                     | Arcturus      | gfx908          |

  使用注意：ROCm 5.7 官方只正式支持 gfx906、gfx908、gfx90a、gfx1030。gfx1031（6700 XT 等）需要手动替换 rocblas 补丁，gfx1100 系列目前不在支持列表。
=======
1. 首次启动 `A绘世启动器.exe` → 点“一键启动”。
2. 弹窗提示缺少 HIP SDK → 弹出官方直链下载 ROCm 5.7 完整版并安装。
3. 6700 XT 需手工补丁，其余6800以下显卡，可对应下载必要库：
   - 下载 `ROCmLibs-gfx1031-5.7.7z`
     https://github.com/likelovewant/ROCmLibs-for-gfx1103-AMD780M-APU/releases
   - 关闭所有调用 ROCm 的程序
   - 备份并替换：
     - `rocblas.dll` → `C:\Program Files\AMD\ROCm\5.7\bin\`
     - 整个 `library` 文件夹 → `C:\Program Files\AMD\ROCm\5.7\bin\rocblas\`
>>>>>>> 90e14e56a8f057db0cc1e5dd8c14cd551cb1629e

## 四、修复 PyTorch

1. 再次打开启动器 → “高级选项” → “环境维护” → 勾选与弹窗提示一致的 PyTorch 版本（2.x for ROCm5.7）。
2. 点击“安装”等待完成。

## 五、首次编译 & 运行

1. 回到首页 → “一键启动”。
   - 首次需 10–30 min 编译 CUDA→HIP 内核，属于正常现象。
2. 当看到 `Running on local URL: http://127.0.0.1:7860` 即部署成功。
3. 浏览器自动打开 WebUI，输入提示词试跑。
   - 第一图较慢（约半小时），第二图起明显加快。

## 六、模型精修

1. 启动器左侧“模型管理”直接拉取 HuggingFace/吐司热门底模；Civitai 需自备代理。
2. 手动放置：`.safetensors` / `.ckpt` 丢到
   `SD-ZLUDA\models\Stable-diffusion`
3. 重启 WebUI → 左上角切换模型 → 再跑图，细节立刻提升。
