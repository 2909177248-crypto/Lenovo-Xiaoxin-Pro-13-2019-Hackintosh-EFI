# Lenovo Xiaoxin Pro 13 (2019) Hackintosh EFI

![macOS Sonoma](https://img.shields.io/badge/macOS-Sonoma%2014.x-blue?style=for-the-badge&logo=apple)
![OpenCore](https://img.shields.io/badge/OpenCore-0.9.x-green?style=for-the-badge)

这是一份为 **联想小新 Pro 13 2019款 (Intel十代)** 精心调优的黑苹果 OpenCore EFI。
经过深度优化，特别针对 **2.5K eDP 高分屏的带宽限制和显存溢出问题** 进行了终极修复，告别花屏与竖线！

## 💻 硬件配置 (Hardware Specs)

| 部件 | 型号 | 状态 |
| :--- | :--- | :--- |
| **CPU** | Intel Core i5-10210U @ 1.60GHz (Comet Lake) | ✅ 原生电源管理，变频正常 |
| **核显** | Intel UHD Graphics 620 | ✅ 完美驱动，2.5K 全分辨率图形加速 (QE/CI) |
| **独显** | NVIDIA GeForce MX250 | ❌ 已通过 `-wegnoegpu` 彻底屏蔽 (省电不发热) |
| **屏幕** | 13.3" 2560x1600 (16:10) | ✅ 亮度调节正常，无开机竖线/黑屏 bug |
| **内存** | 16GB LPDDR3 2133MHz | ✅ 识别正常 |
| **固态硬盘** | 忆联 (UMIS) LENSE40256GMSP34MESTB3A NVMe 256GB | ✅ `NVMeFix` 驱动，支持 APST 省电温度控制 |
| **网卡/蓝牙** | Intel Wireless-AC 9462 (en0) / 蓝牙 | ✅ itlwm驱动完美支持，可连WiFi及蓝牙外设 |
| **声卡** | Realtek ALC257 | ✅ 扬声器/耳机/麦克风正常 |
| **触控板** | I2C 触控板 | ✅ 支持原生多指手势 |
| **电池** | 56Wh 锂电池 | ✅ 电量显示正常 |


## 🛠️ 核心调优说明 (针对 2.5K 屏幕与机身特性的深度修复)

### 1. 2.5K 屏幕显卡补丁 (解决花屏与黑屏)
由于十代移动端 UHD 620 驱动 2560x1600 eDP 屏幕存在先天带宽握手问题，本 EFI 进行了如下显卡深度修补：
*   **强制解锁链路带宽**：注入 `dpcd-max-link-rate` 为 `0A000000` (HBR2 速率)，配合 `enable-max-pixel-clock-override` 彻底解决开机花屏与竖线。
*   **显存防溢出机制**：注入 `framebuffer-stolenmem` (`48MB`) 与 `framebuffer-fbmem` (`9MB`)，绕过 BIOS 限制，防止显存溢出导致 WindowServer 崩溃死机。
*   **背光与输出修复**：添加 `-igfxblr` 启动参数解决切换分辨率黑屏，并使用 `ClearScreenOnModeSwitch` 消除开机联想 Logo 闪烁残影。

### 2. 固态硬盘功耗与降温 (`NVMeFix.kext` 注入)
*   针对原装忆联 (UMIS) NVMe 固态硬盘，集成了 `NVMeFix.kext`。
*   开启自主电源状态转换 (APST)，大幅**降低硬盘运行温度 5~10°C**，增强读写寿命并明显改善整机电池续航。

### 3. 合盖睡眠深度省电优化 (解决休眠掉电)
通过重构 macOS 电源管理参数，解决黑苹果睡眠频繁唤醒和异常掉电：
```bash
sudo pmset -a hibernatemode 0 proximitywake 0 autopoweroff 0 standby 0 tcpkeepalive 0
```
*   将休眠模式设为 `0`（内存挂起，实现 1 秒极速唤醒）。
*   关闭 `proximitywake` 和 `tcpkeepalive`，彻底断开合盖后的后台多余网络同步，**实现合盖数天仅掉电 1~2%**。

### 4. 2.5K Retina HiDPI 视网膜清晰度缩放
*   内置屏幕原生分辨率直接使用会导致字号极小。本仓库推荐使用 **BetterDisplay** 或运行 `one-key-hidpi` 开启“平滑缩放”。
*   推荐在设置中选用 **`1440x900 (HiDPI)`** 档位，获得白苹果 13.3 英寸同等极致细腻的视网膜超清视觉体验。


## ⚠️ 使用前的注意事项

1. **生成自己的三码 (SMBIOS)**：
   为了保护隐私和账号安全，本 EFI 已经抹除了序列号 (三码)。**使用前请务必使用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 或 OpenCore Configurator 重新生成一套属于你自己的 `MacBookPro16,3` 的三码。**
   需要填写的字段位于 `config.plist` -> `PlatformInfo` -> `Generic`：
   - `SystemSerialNumber`
   - `MLB`
   - `SystemUUID`

2. **每次更换 EFI 后，务必重置 NVRAM！**
   在 OpenCore 引导菜单界面（如果是图形界面，按空格键显示隐藏选项），一定要选中 **`Reset NVRAM`** 并回车。否则新的显存参数不会生效，大概率会花屏。

## 📥 如何使用
1. 下载整个 `EFI` 文件夹。
2. 使用工具（如 DiskGenius）替换你安装 U 盘或笔记本内置硬盘 EFI 分区中的 `EFI` 文件夹。
3. 补齐你的 SMBIOS 三码。
4. 重启选择引导，**重置 NVRAM**，然后享受你的黑苹果！



## 🙏 致谢
感谢原版 EFI 作者“乌龙蜜桃来一打”，以及 Acidanthera 团队提供的 OpenCore 和各类强大的 kext 驱动。
