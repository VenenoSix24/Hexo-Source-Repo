---
title: AMD 平台福音：微星 B350M 迫击炮主板黑苹果实践指南 (macOS Sequoia)
date: 2025-08-16 17:22:00
categories: 实用教程
tags:
  - OpenCore
  - EFI
  - macOS
  - 黑苹果
  - 教程
cover: https://s2.loli.net/2025/08/13/xZ5RYIHwJyMTSoe.png
---

对于许多 AMD 平台的 PC 爱好者来说，在自己的高性能 Ryzen 主机上运行流畅、稳定的 macOS 一直是一个令人向往的目标。得益于 OpenCore 引导工具和社区开发者的不懈努力，这个目标早已从不可能变为现实。

今天，我将分享专为经典主板 **微星 B350M 迫击炮 (MSI B350M MORTAR)** 打造的 OpenCore EFI 配置，它针对 **Ryzen 5000 系列处理器** 与 **Navi 2X 系列显卡** 进行了深度优化，让您可以在 AMD 平台上体验到接近“开箱即用”的 macOS Sequoia。

**使用机型：** `MacPro7,1` 
**OpenCore 版本：** `1.0.5` 
**最高支持：** `macOS Sequoia 15.7` 

![2025-08-13 23.16.11_compressed.png](https://s2.loli.net/2025/08/13/xZ5RYIHwJyMTSoe.png)

## 我的硬件配置单

这份 EFI 是基于我自己的主力硬件配置进行制作和测试的，理论上，采用同代 Ryzen 处理器和 Navi 核心显卡的配置都有很高的参考价值。

|硬件|型号|
|---|---|
|主板|微星 B350M MORTAR|
|处理器|AMD Ryzen™ 5 5600|
|显卡|AMD Radeon RX 6750 GRE 12GB|
|内存|玖合 星舞 32G 3200MHz DDR4|

## 功能实现状态一览

经过细致的配置和测试，这套 EFI 在日常使用中的表现相当完善。以下是各项主要功能的当前状态，供您参考：

| 功能                       | 状态     | 备注                                         |
| ------------------------ | ------ | ------------------------------------------ |
| ✅ **GPU 图形加速**           | `正常`   | Navi 核心显卡硬件加速与 H.264/HEVC 视频硬解均可正常工作。      |
| ✅ **板载声卡 / 网卡**          | `正常`   | 分别由 `AppleALC` 和 `RealtekRTL8111` 驱动，工作稳定。 |
| ✅ **App Store / iCloud** | `正常`   | 正确生成并配置三码后，可完美登录和使用苹果生态服务。                 |
| ✅ **HDMI 音视频输出**         | `正常`   | 即插即用，无需额外配置。                               |
| ✅ **睡眠与唤醒**              | `正常`   | 如果遇到唤醒异常，通常需要重新定制 USB 端口映射。                |
| ✅ **USB 端口**             | `正常`   | EFI 已包含基础的 USB 映射，但强烈建议您根据自己的机箱接口重新定制一次。   |
| ✅ **Apple Music**        | `半正常`  | 开启无损音质会导致没有声音输出，这是待解决的问题。                  |
| ❌ **Apple TV+**          | `无法使用` | AMD 平台常见的 DRM 版权保护问题，暂时无解。                 |
| ❓ **蓝牙 / Wi-Fi**         | `未测试`  | 主板本身不带无线模块，因此无法测试。                         |

## 安装前的准备：BIOS 设置

正确的 BIOS 设置是成功安装的第一步。在开始之前，请确保您的 BIOS 配置如下（版本参考: `7A37v1O7`）：

|选项|状态|
|---|---|
|SATA Mode|选择 AHCI|
|Above 4G Decoding|禁用 / Disabled|
|EHCI/XHCI Hand-off|启用 / Enabled|
|SVM|启用 / Enabled|
|CSM|禁用 / Disabled|
|Secure Boot|禁用 / Disabled|
|Serial Port|禁用 / Disabled|
|Parallel Port|禁用 / Disabled|

> **特别注意**：
> 
> - 关于 `Above 4G Decoding` 选项，您必须在 **“禁用此项”** 和 **“删除 `boot-args` 中的 `npci=0x3000` 参数”** 之间二选一。本 EFI 默认保留了 `npci=0x3000` 参数，因此请在 BIOS 中禁用 `Above 4G Decoding`。
>     
> - 根据测试，如果开启 `Above 4G Decoding` 并删除 `npci=0x3000`，同时 `ResizeAppleGpuBars` 设置为 `0`，会导致睡眠唤醒后重启系统。
>     

## 快速上手指南：三步配置你的专属 EFI

详细的 macOS 安装过程可以参考社区的优秀教程，这里我们重点讲解如何配置这份 EFI 文件。

### 第一步：下载并修改 CPU 核心数

1. 前往项目的 **[Releases 页面](https://github.com/VenenoSix24/MSI-B350M-MORTAR-Hackintosh-OpenCore1.0.2-EFI/releases)** 下载最新版 EFI。
    
2. 使用 Plist 编辑器（如 [OCAuxiliaryTools](https://github.com/ic005k/OCAuxiliaryTools) 或 [OpenCore Configurator](https://mackie100projects.altervista.org/opencore-configurator/)）打开 `EFI/OC/config.plist` 文件。
    
3. 导航到 `Kernel -> Patch` 路径，找到四个 `algrey - Force cpuid_cores_per_package` 开头的补丁。
    
4. 根据您 CPU 的物理核心数，参考下表修改补丁中的十六进制值。例如，我的 R5 5600 是 6 核心，就将 `00` 修改为 `06`。
    
    - `B8 00 0000 0000` -> `B8 06 0000 0000`
        
    - `BA 00 0000 0000` -> `BA 06 0000 0000`
        
    - `BA 00 0000 0090` -> `BA 06 0000 0090`
        
    - `BA 00 0000 00` -> `BA 06 0000 00`
        

| CPU 核心 | 十六进制值 |
|:--------:|:----------:|
|   4 核   |    `04`    |
|   6 核   |    `06`    |
|   8 核   |    `08`    |
|  12 核   |    `0C`    |
|  16 核   |    `10`    |
|  24 核   |    `18`    |
|  32 核   |    `20`    |

### 第二步：生成并替换“三码”

**这是让 iCloud、iMessage 等苹果服务正常工作的关键一步，请务必操作！**

1. 使用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 或其他工具，选择 `Generate SMBIOS`。
    
2. 机型选择 `MacPro7,1` 来生成您自己的序列号、主板序列号和 SmUUID。
    
3. **验证序列号**：访问 [苹果官方查询页面](https://checkcoverage.apple.com/?locale=zh_CN)，粘贴生成的序列号进行查询。如果结果是“无效序列号”，那么这个序列号就是可用的。
    
4. 将生成的三个值，依次填入 `config.plist` 文件的 `PlatformInfo -> Generic` 路径下。
    

### 第三步：开始安装

将您配置好的 `EFI` 文件夹完整复制到 macOS 安装 U 盘的 ESP 分区中，然后就可以从 U 盘引导开始安装了。具体的安装步骤可以参考这个[视频教程](https://www.bilibili.com/video/BV1Ps421M7NZ)。

## 安装后的优化与调试

为了让系统更完美，一些安装后的调优是必不可少的。

### 1. 修复睡眠

如果您的系统无法正常睡眠或唤醒，通常是 USB 端口问题。请参考 **[国光的 USB 定制教程](https://apple.sqlsec.com/6-%E5%AE%9E%E7%94%A8%E5%A7%BF%E5%8A%BF/6-1/)** 来创建您自己的 USB 映射。若问题依旧，可参考 **[官方睡眠修复文档](https://dortania.github.io/OpenCore-Post-Install/universal/sleep.html)**。

### 2. 开启 HiDPI

为了获得更细腻的“Retina”显示效果，可以使用 **[one-key-hidpi](https://github.com/xzhih/one-key-hidpi)** 项目的一键脚本来开启。在终端中运行以下命令，按提示操作即可：

```Bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/xzhih/one-key-hidpi/master/hidpi.sh)"
```

### 3. PAT 补丁切换

本 EFI 默认启用 `Shaneee` 的 PAT 补丁以获得更好的 GPU 性能。如果您遇到 DP/HDMI 音频输出问题或使用的是 NVIDIA 显卡，可以切换到兼容性更好的 `Algrey` 补丁。

- **路径**：`config.plist` -> `Kernel` -> `Patch`
    
- **操作**：搜索 `mtrr_update_action`，禁用 Shaneee 的补丁并启用 Algrey 的。**切勿同时启用两个！**
    
|**Shaneee's**|**Algrey's**|
|---|---|
|更好的 GPU 性能|标准性能/兼容模式|
|可能不适用于 NVIDIA GPU|兼容所有 GPU|
|HDMI / DP 音频可能无法工作|HDMI / DP 音频正常工作|
|✅ **默认启用**|❌ **默认禁用**|

### 4. Adobe 及 MKL 程序修复

部分 Adobe 应用和依赖 Intel MKL 库的程序在 AMD 平台上无法正常运行。

- **Adobe 修复**：运行 **[此修复脚本](https://github.com/VenenoSix24/MSI-B350M-MORTAR-Hackintosh-OpenCore-EFI/tree/main/Resource/adobe_patch.sh)**。
    
- **MKL 修复**：运行 **[此修复脚本](https://github.com/VenenoSix24/MSI-B350M-MORTAR-Hackintosh-OpenCore-EFI/tree/main/Resource/ryzen_patch.sh)**。
    
- 运行后**重启**查看效果。
    
### 5. 修改 CPU 处理器名称

您可以在 `config.plist` 的 `NVRAM -> 4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102 -> cpu-name` 项中修改成任何您喜欢的名字，修改后重启并重置 NVRAM 即可生效。

## 如何更新 EFI？

1. **备份**当前能正常工作的 EFI 文件夹，尤其是您的 `config.plist` 文件。
    
2. 下载最新的 Release 版本。
    
3. 使用 `OCAuxiliaryTools` 等工具的对比功能，将您旧 `config.plist` 中的个人设置（如三码、CPU核心数）迁移到新版 `config.plist` 中。
    
4. 替换整个 EFI 文件夹，重启并重置 NVRAM。
    
## 常见问题 (FAQ)

Q: 安装后没有声音怎么办?

A:

1. 检查 `config.plist` -> `NVRAM` -> `boot-args` 中是否包含 `alcid=xx` 参数。对这块主板，`alcid=1` 或 `alcid=7` 通常是有效的。
    
2. 前往 `系统设置 -> 声音 -> 输出`，确认输出设备是否已正确选择。
    

Q: 更新 Kexts 或 OpenCore 后无法启动了?

A: 立即使用备份好的旧版 EFI 恢复系统。然后仔细排查新旧配置文件的差异，确保所有 Kexts、驱动和 OpenCore 版本相互兼容。

## 更新日志

**v2025.08.14**

- 更新 OpenCore 版本至: v1.0.5
    
- 系统支持：最高支持到 `macOS Sequoia 15.7`
    
- 默认启用图形化 OC 引导界面，主题位于 `Resources` 文件夹下
    
- Kexts 均更新至最新版，为追求稳定， `AppleALC` 依旧使用 v1.9.2 修改版
    
- 由于 Tahoe Beta 版稳定性问题，暂未在物理机上测试，所以，**对 macOS 26 Tahoe 的支持性未知**

**v2025.08.13**

- 当前 OpenCore 版本: v1.0.2
    
- 系统支持：最高支持到 `macOS Sequoia 15.6`
    
- **注意:** 默认添加 `npci=0x3000`参数，请关闭 `Above 4G Decoding`选项（或删除 `npci=0x3000`参数）
    

**v2025.05.21**

- 删除不必要的文件
    
- 更新 Lilu 和 NootRX
    
- 调整 config.plist 配置
    
- 更新 README.md 文档
    

**v2024.10.21**

- OpenCore 升级至 1.0.2 版本
    
- 同步更新所有 Kexts 至最新版
    
- 针对 macOS Sequoia 15 进行初步适配
    

## 鸣谢与资源

> 感谢所有为 Hackintosh 社区做出贡献的开发者和先行者！

- **教程与社区**: [国光酱](https://apple.sqlsec.com/), [黑果小兵](https://blog.daliansky.net/), [Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)
    
- **核心项目**: [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg), [AMD_Vanilla](https://github.com/AMD-OSX/AMD_Vanilla)
    
- **工具**: [ProperTree](https://github.com/corpnewt/ProperTree), [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS), [OCAuxiliaryTools](https://github.com/ic005k/OCAuxiliaryTools), [OpenCore Configurator](https://mackie100projects.altervista.org/opencore-configurator/)
    
- **补丁与脚本**: [ryzen-hackintosh](https://github.com/mikigal/ryzen-hackintosh), [one-key-hidpi](https://github.com/xzhih/one-key-hidpi)
    

## **重要声明**

本项目仅为技术爱好者学习和研究之用，所有文件均来自网络。使用者需自行承担因使用此 EFI 配置所带来的任何风险，包括但不限于数据丢失、硬件损坏、法律纠纷等。本人不对任何可能产生的问题负责。

**在进行任何操作前，请务必备份你的所有重要数据！**