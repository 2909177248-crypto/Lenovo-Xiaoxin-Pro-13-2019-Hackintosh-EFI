# 联想小新 Pro 13 2019 Intel款 黑苹果 (macOS Sonoma) 完整配置与优化技术文档

本技术文档详尽记录了针对 **联想小新 Pro 13 2019款 (Intel十代)** 配置黑苹果 macOS Sonoma 系统的硬件参数、引导配置（OpenCore）、核心设备补丁以及系统调优策略。

---

## 1. 💻 硬件规格与驱动状态 (Hardware Specifications)

| 硬件部件 | 具体型号 / 规格 | 驱动 Kext / 注入补丁 | 当前状态 |
| :--- | :--- | :--- | :--- |
| **CPU** | Intel Core i5-10210U (Comet Lake, 4核8线程) | 原生 CPU 电源管理 (SMCProcessor) | ✅ 变频极佳，睿频正常 |
| **集成显卡** | Intel UHD Graphics 620 (彗星莱克核显) | WhateverGreen + DeviceProperties 显存/带宽注入 | ✅ 完美驱动，支持 QE/CI 图形加速 |
| **独立显卡** | NVIDIA GeForce MX250 | `-wegnoegpu` 引导参数屏蔽 | ❌ 彻底关闭 (省电、防发热) |
| **内置屏幕** | 13.3" IPS 2560x1600 (16:10, 2.5K 高分屏) | `dpcd-max-link-rate` 补丁 + `BetterDisplay` | ✅ 亮度调节顺滑，支持 Retina HiDPI 缩放 |
| **固态硬盘** | 忆联 (UMIS) LENSE40256GMSP34MESTB3A NVMe | `NVMeFix.kext` | ✅ 开启 APST 自主电源管理，降温 5~10°C |
| **无线网卡** | Intel Wireless-AC 9462 (或自行升级的板载网卡) | `AirportItlwm.kext` (Sonoma 专版) | ✅ 原生 Wi-Fi 连接，系统菜单正常控制 |
| **蓝牙** | Intel Wireless Bluetooth | `IntelBluetoothFirmware` + `BlueToolFixup` | ✅ 蓝牙外设连接正常 |
| **声卡** | Realtek ALC257 | `AppleALC.kext` (layout-id: `11` 或 `21`) | ✅ 扬声器、耳机插拔、麦克风正常 |
| **触控板** | 联想 I2C 触控板 | `VoodooI2C.kext` + `VoodooI2CHID.kext` | ✅ 支持原生 macOS 多指手势 |
| **电池管理** | 56Wh 三芯锂离子电池 | `ECEnabler.kext` + `SMCBatteryManager.kext` | ✅ 电量百分比显示精准，估算时间正常 |

---

## 2. ⚙️ BIOS 核心设置 (BIOS Settings)

进入 BIOS（开机狂按 `Fn` + `F2`），进行以下修改：



*   **SATA Controller Mode**: 设置为 `AHCI`（不要使用 RAID/RST，否则 macOS 无法识别硬盘）。
*   **Secure Boot**: 设置为 `Disabled`（关闭安全启动，否则无法引导 OpenCore）。
*   **Fast Boot**: 设置为 `Disabled`（关闭快速启动，防止引导冲突）。
*   **Intel Virtualization Technology (VT-x)**: 设置为 `Enabled`。
*   **VT-d**: 设置为 `Enabled`（在 `config.plist` 中已开启 `DisableIoMapper`，保持开启不影响 Windows）。
*   **DVMT Pre-Allocated**: 保持主板默认配置（32MB）。由于我们使用了 stolenmem 显存补丁，**无需解密或刷写 BIOS 解锁 64MB DVMT**。

---

## 3. 🛠️ OpenCore 核心配置参数 (OpenCore Key Configs)

### 3.1 显卡与屏幕参数注入 (`DeviceProperties`)
这是解决 **2.5K 屏幕开机闪烁、花屏、死机** 的最核心配置。在 `config.plist` -> `DeviceProperties` -> `Add` -> `PciRoot(0x0)/Pci(0x2,0x0)` 注入：

*   **AAPL,ig-platform-id**: `00009B3E` (十代移动端核显帧缓冲区布局)
*   **device-id**: `9B3E0000` (强制识别为 UHD 620)
*   **framebuffer-patch-enable**: `01000000` (开启显卡修补)
*   **framebuffer-stolenmem**: `00003001` (注入 **19MB stolenmem**，限制在主板默认 32MB 以下，防止 WindowsServer 崩溃)
*   **framebuffer-fbmem**: `00009000` (注入 **9MB fbmem** 作为帧缓冲缓存)
*   **dpcd-max-link-rate**: `0A000000` (强制锁死屏幕 eDP 链路速率为 **HBR2**，防止与 2.5K 屏握手失败导致开机黑屏/竖线)
*   **enable-max-pixel-clock-override**: `01000000` (突破最大像素时钟限制)
*   **enable-dpcd-max-link-rate-fix**: `01000000` (启用链路速率修复补丁)

### 3.2 引导参数 (`boot-args`)
在 `config.plist` -> `NVRAM` -> `Add` -> `7C436110-AB2A-4BBB-A880-FE41995C9F82` -> `boot-args` 中：

```text
revpatch=sbvmm -wegnoegpu keepsyms=1 forceRenderStandby=0 debug=0x100 -igfxblr igfxonln=1
```
*   `-wegnoegpu`：彻底屏蔽独显 MX250，极大延长续航、降低机身温度。
*   `-igfxblr`：修复十代 UHD 620 在引导进入 macOS 桌面、分辨率切换时黑屏的 Bug。
*   `igfxonln=1`：强制所有核显输出端口在线。
*   `keepsyms=1` / `debug=0x100`：保留调试符号，方便崩溃（Kernel Panic）时排错。

---

## 4. 🚀 系统级后期优化 (Post-Install Optimizations)

### 4.1 完美的深度睡眠配置 (解决休眠电池掉电)
黑苹果默认休眠会因为后台网络同步（Proximity Wake）和频繁自检导致电量在合盖后迅速耗尽。在终端运行以下命令：

```bash
sudo pmset -a hibernatemode 0 proximitywake 0 autopoweroff 0 standby 0 tcpkeepalive 0
```
*   `hibernatemode 0`：将休眠模式设置为普通睡眠（挂载到内存），休眠和唤醒速度极快（1秒）。
*   `proximitywake 0` / `tcpkeepalive 0`：关闭网络保持活动状态。**彻底解决合盖掉电，合盖三天仅掉电 1~2%**。

### 4.2 2.5K Retina HiDPI 视网膜超清缩放
由于 2560x1600 分辨率直接使用会导致字太小，使用 **BetterDisplay** 工具开启平滑缩放（Flexible Scaling）：
1. 在 BetterDisplay 设置中，找到内置显示器（Built-in Display）。
2. 勾选 **"Edit the default system configuration of this display model"**（编辑此显示器型号的默认系统配置）。
3. 勾选 **"Enable flexible scaling"**（启用平滑缩放 / 弹性分辨率）。
4. 点击 **Apply** 并输入密码重启。
5. 重启后在菜单栏一键选择 **`1440x900 (HiDPI)`**，这是 13.3 寸屏幕的黄金视网膜缩放比例，清晰度极高，字号最舒适。

### 4.3 屏蔽 Windows 启动项，净化开机界面
如果内置硬盘装有单盘双系统，为防止 OpenCore 菜单里多余的 Windows 启动项显得杂乱：
*   在将引导迁移至内置硬盘 EFI 分区时，**不复制** 微软引导文件夹（`EFI/Microsoft`），仅保留 `EFI/BOOT` 与 `EFI/OC`。
*   或者在 `config.plist` -> `Misc` -> `Boot` -> `HideAuxiliary` 设置为 `true`。

### 4.4 消除 BIOS 开机 Logo 闪烁残留
在 `config.plist` -> `UEFI` -> `Output` 中：
*   将 **`ClearScreenOnModeSwitch`** 设置为 `true`。
*   这会促使 OpenCore 在进行图形显示模式切换时立刻清屏，彻底消灭联想小新开机 Logo 的残影，直接无缝过渡到纯净的 Apple Logo 进度条。

---

## 5. 🔑 隐私脱敏与 SMBIOS 三码生成 (Privacy & SMBIOS)

本 EFI 在开源或共享前，必须清空 SMBIOS 信息。使用者在正式引导前，需要自行生成三码：

1. 使用 **GenSMBIOS** 工具，机型（Product Name）选择 **`MacBookPro16,2`** 或 **`MacBookPro16,3`**。
2. 将生成的数据填入 `config.plist` 的 `PlatformInfo -> Generic`：
   *   `SystemSerialNumber` (系统序列号)
   *   `MLB` (主板序列号)
   *   `SystemUUID` (系统唯一识别码)
3. 填入后保存，重启进入系统前务必在 OpenCore 界面执行 **Reset NVRAM**！
