# Lenovo Xiaoxin Pro 13 (2019) Hackintosh EFI · 双版本终极完美调优版

[![macOS Sonoma](https://img.shields.io/badge/macOS-Sonoma%2014.x-blue?style=for-the-badge&logo=apple)](https://www.apple.com/macos/)
[![macOS Ventura](https://img.shields.io/badge/macOS-Ventura%2013.x-purple?style=for-the-badge&logo=apple)](https://www.apple.com/macos/)
[![OpenCore](https://img.shields.io/badge/OpenCore-0.9.x-green?style=for-the-badge)](https://github.com/acidanthera/OpenCorePkg)
[![License](https://img.shields.io/badge/License-MIT-orange?style=for-the-badge)](LICENSE)

专为 **联想小新 Pro 13 2019款 (Intel十代Comet Lake平台)** 打造的深度调优黑苹果 OpenCore 引导配置。

本项目已全面重构并**独立分离为两大开箱即用版本**：
1. **🟢 原生 Intel 网卡开箱即用版**：专为未拆机原装网卡用户打造，驱动成熟稳定；
2. **🍎 苹果博通 BCM94360 白苹果满血版**：专为更换免驱卡（如 BCM94360Z4 / CS2 / NG）用户打造，彻底根治总线挂死与死锁！

已彻底攻克小新 Pro 13 长期以来的三大经典神坑：
- 🛡️ **博通网卡 PCIe ASPM 总线挂死与硬死机修复**
- 🖥️ **2.5K 高分屏 `forceRenderStandby=0` 顶部偶发跳闪与绿线死锁修复**
- 🚀 **eDP 链路 DPCD 带宽解锁与 32MB DVMT 防显存溢出**

---

## 💻 硬件配置与支持清单 (Hardware Specs)

| 部件 | 硬件型号 / 规格 | 驱动状态 | 调优说明 |
| :--- | :--- | :---: | :--- |
| **处理器 (CPU)** | Intel Core i5-10210U / i7-10710U (Comet Lake) | ✅ 完美 | 原生 X86 电源管理，多级智能睿频与功耗释放 |
| **核芯显卡 (iGPU)** | Intel UHD Graphics 620 (伪装 CFL 0x3E9B) | ✅ 完美 | 满血 Metal 3、硬件加速 (QE/CI)、4K/2.5K 硬解 |
| **独立显卡 (dGPU)** | NVIDIA GeForce MX250 | ❌ 屏蔽 | 已通过 `-wegnoegpu` 彻底屏蔽底层供电，凉爽省电 |
| **内建屏幕** | 13.3\" 2560x1600 (16:10) 华星光电/友达 2.5K | ✅ 完美 | 注入 DPCD 解锁带宽；注入 `forceRenderStandby=0` 防闪屏 |
| **内存 (RAM)** | 8GB / 16GB 双通道 DDR4 2666MHz | ✅ 完美 | 原生识别双通道 |
| **固态硬盘 (SSD)** | 忆联 (UMIS) / 三星 PM981a 等 NVMe SSD | ✅ 完美 | 集成 `NVMeFix.kext` 开启 APST 省电控制，降温 5~10°C |
| **声卡 (Audio)** | Realtek ALC257 (Layout-ID: 99) | ✅ 完美 | 扬声器、3.5mm 耳机自动切换、内建阵列麦克风正常 |
| **触控板 (Trackpad)** | I2C HID 触控板 (GPIO 中断模式) | ✅ 完美 | 支持原生 macOS 1~4 指丝滑手势 |
| **电池与电源管理** | 56Wh 原装锂电池 + YogaSMC | ✅ 完美 | 睡眠合盖数天掉电仅 1~2%，支持键盘快捷键调节 |
| **原装无线网卡** | Intel AX201 (CNVi) / AC9560 / AC9462 | ✅ 正常 | 使用 `EFI-Intel-Default`，支持 Wi-Fi 与普通蓝牙连接 |
| **白苹果免驱卡** | 博通 BCM94360Z4 / BCM94360CS2 等 | 🍎 极品 | 使用 `EFI-Broadcom-BCM94360`，解锁隔空投送、随航、接力 |

---

## 🗂️ 双版本 EFI 目录架构与选型指南

仓库提供两大独立分支，根据你的硬件配置**直接复制对应目录**：

```bash
Lenovo-Xiaoxin-Pro-13-2019-Hackintosh-EFI/
├── 📁 EFI-Broadcom-BCM94360/       # 方案 A：博通白苹果免驱卡专属版本 (推荐换卡用户)
│   └── EFI/
│       ├── BOOT/
│       └── OC/
│           ├── config.plist         # 已集成 ASPM=0、forceRenderStandby=0、IOSkywalk 降级
│           ├── ACPI/
│           ├── Drivers/
│           └── Kexts/
│
├── 📁 EFI-Intel-Default/           # 方案 B：原装 Intel 无线网卡专属版本 (免拆机用户)
│   └── EFI/
│       ├── BOOT/
│       └── OC/
│           ├── config.plist         # 已启用 AirportItlwm + Intel 蓝牙套件 + forceRenderStandby=0
│           ├── ACPI/
│           ├── Drivers/
│           └── Kexts/
│
├── 📁 EFI/                         # 快捷默认入口 (与 EFI-Broadcom-BCM94360 保持同步)
├── 📄 使用说明_新手必读.md           # 详细部署、换卡、OCLP 打补丁与排障手册
└── 📄 README.md                    # 本文档
```

### 两种版本怎么选？
* **选 `EFI-Intel-Default`**：
  如果你**没有拆开后盖更换无线网卡**，使用的是出厂自带的 Intel 网卡。
  - 支持连接 Wi-Fi 与蓝牙外设（蓝牙音箱、鼠标）；
  - 缺点：不支持隔空投送 (AirDrop)、随航 (Sidecar)、通用控制等苹果私有连续互通功能。
* **选 `EFI-Broadcom-BCM94360`**：
  如果你拆机更换了 **BCM94360Z4 / BCM94360CS2 / BCM94360NG** 等博通原生苹果网卡。
  - **满血体验**：原生隔空投送 (AirDrop)、随航 (Sidecar)、通用控制 (Universal Control)、接力 (Handoff)、Apple Watch 自动解锁；
  - **安全保障**：已禁用 `Pci(0x1C,0x0)` 桥的 ASPM 节能，彻底消灭数据流量导致的 PCIe 总线挂死与硬死机。

---

## 🌟 本项目的重大核心突破与调优技术

### 1. 彻底根治博通卡「一用 Wi-Fi 就瞬间死机 / 绿线」
* **问题本质**：小新 Pro 13 的 M.2 网卡槽位于 PCIe 桥 `RP05` (`Pci(0x1C,0x0)`)。原机针对低功耗开启了 `pci-aspm-default = 0x02` (L1) 与 `0x0102` (L0s+L1)。博通白苹果卡对该节能时序极度敏感，一旦有网络流量进出，芯片在 L1 切换瞬间直接拉死 PCIe 总线，导致显存管线失步拉绿线并硬卡死。
* **终极解决**：在 DeviceProperties 中注入 `pci-aspm-default = <00 00 00 00>`，完全关闭该网卡槽的 ASPM，**连续满速测速与大文件传输 100% 稳定不卡死**！

### 2. 攻克 2.5K 屏幕「屏幕上方偶发跳闪 (Top Screen Flickering)」
* **问题本质**：换卡后遗漏了关键启动参数 `forceRenderStandby=0`。Intel 核显在驱动 2.5K 屏幕时频繁切入 RC6（Render Standby 渲染深度休眠），鼠标滑动或窗口变化瞬间唤醒加压产生时钟毛刺，导致屏幕顶端 1~2 像素发生横向跳闪。
* **终极解决**：注入 `forceRenderStandby=0`，平息时钟毛刺，画面稳如泰山。

### 3. 解锁 2.5K eDP 链路带宽与 32MB DVMT 防爆
* 注入 `dpcd-max-link-rate = 0A000000` (HBR 速率) 与 `enable-max-pixel-clock-override`，解决开机撕裂；
* 注入 `framebuffer-stolenmem = 19MB` 与 `framebuffer-fbmem = 9MB`，使动态显存严格落在主板 32MB DVMT 预分配范围内，从根本上防止显存爆仓黑屏。

---

## 🚀 快速使用步骤

### 第一步：注入你的专属三码 (SMBIOS)
为防止 Apple ID 被封禁并保证 iMessage / FaceTime 正常，请务必生成专属三码：
1. 下载 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)；
2. 选择机型 **`MacBookPro16,3`**；
3. 打开对应版本中的 `EFI/OC/config.plist`，在 `PlatformInfo` -> `Generic` 下填入：
   - `SystemSerialNumber`
   - `MLB`
   - `SystemUUID`

### 第二步：部署到 EFI 分区
1. 使用 OpenCore Configurator、OCC 或终端挂载硬盘 EFI 分区：
   ```bash
   sudo diskutil mount /dev/disk0s1
   ```
2. 将对应版本中的 `EFI` 文件夹完整拷贝到磁盘根目录。

### 第三步：开机重置 NVRAM
1. 重启电脑，在 OpenCore 引导选单界面按 **空格键** 显示隐藏项；
2. 选中 **`Reset NVRAM`** 并回车（电脑会自动重启一次）；
3. 随后正常进入系统即可！

---

## 🛠️ macOS 14 Sonoma 博通卡网络驱动指南 (仅限博通版)
macOS Sonoma 移除了对旧款博通卡的原生驱动，使用 `EFI-Broadcom-BCM94360` 需配合 OCLP 激活：
1. 正常进入系统（可先使用手机 USB 共享网络）；
2. 打开安装 [OpenCore-Patcher (OCLP)](https://github.com/dortania/OpenCore-Legacy-Patcher)；
3. 点击 **`Post-Install Root Patch`** -> 点击 **`Start Root Patching`**；
4. 提示完成后重启电脑，Wi-Fi 与隔空投送立刻满血复活！

---

## 📋 推荐系统分辨率设置
小新 Pro 13 物理分辨率为 2560x1600 (16:10)，推荐在系统中选择：
* 🥇 **`1440 x 900 (HiDPI)`（最推荐，黄金平衡档）**：类似 13.3 寸 MacBook Pro 默认视觉大小，字体最舒适、窗口空间宽敞，可通过系统已常驻的 BetterDisplay 自由滑到此档；
* 🥈 **`1280 x 800 (HiDPI 2x)`（点对点极致清晰档）**：物理 2x 整数倍渲染，字体最黑最锐利，零模糊，可在「系统设置 -> 显示器」直接点选。

---

## 📄 开源许可与致谢
* [Acidanthera](https://github.com/acidanthera) 提供 OpenCore 及全套 Lilu、WhateverGreen、VirtualSMC 核心套件；
* [Dortania](https://dortania.github.io/) 团队提供 OCLP 与权威构建指南；
* 乌龙蜜桃来一打及小新黑苹果社区各位先锋探路者。
