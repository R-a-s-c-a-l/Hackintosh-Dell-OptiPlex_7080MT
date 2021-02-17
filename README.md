# Hackintosh-OptiPlex-7080-MT

## 简介
**Dell OptiPlex 7080MT EFI**， 支持**macOS Catalina 10.15.5** 或更高版本

## 硬件
* **CPU**: Intel® Core™ i5-10500
* **IGPU**: Intel® UHD Graphics 630
* **RAM**: 8GB DDR4 2666 Daul Channel
* **SSD**: APPLESSD SM0128L Media & Samsung SSD 860 EVO 
* **Wi-Fi & Bluetooth**: BCM943602CDP

## 可用
* CPU 睿频
* IGPU 图形加速，HEVC H.264 En/Decode 
* ALC256 所有输入输出正常 LayoutID 67 线路输出独立，可选内置扬声器与耳机 或 线路输出与耳机自动切换麦克风工作需要VerbStub和ComboJack (AppleALC 已合并)
![演示.png](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/演示.gif)
* 除最靠近网口的USB2.0(为仅保留15端口而屏蔽)其他的正常
* 以太网卡 
* 睡眠唤醒

## 补充及说明
* **USB端口定制** EFI使用空壳Kext 基于7080MT定制，`SMBIOS iMac19,2`

* **蓝牙** 不同的网卡的安装方式可能蓝牙HCI位置可能不同，我的蓝牙USB跳线直接插在主机后的第二个`USB2.0 Type-A` 端口 (`HS09`) 

* **传感器方面** 相关驱动请自行添加

* **WhateverGreen** 使用的精简版 仅包含核心显卡`GFX0 → IGPU `，管理引擎` HECI → IMEI`的重命名以及注入` Metal Device Name` (仅支持`FakeID 3E9B`) 有独立显卡的请自行更换原版

* **IOKitPersonalitiesInjector** 空壳驱动 包含 iMac19,2 SMBIOS 仅核显的`AGPM`注入来修复核显视频处理时满载满频且不保留基本图形渲染性能而导致UI卡顿的问题 USB 定制 修改系统报告 SATA 控制器的名称 GenericAHCI → Intel 14 Series Chipset (仅作修饰)，以及USB电流属性注入 (仅针对11.0 的新`IOProviderClass：AppleUSBHostResources`，之前版本为`AppleBusPowerController`)
可为 iOS 设备提供12W的充电功率 (5v 2.4A)

* **DVMT预分配和CFGLock** EFI中不包含DVMT补丁请自行使用 `Ru.efi` 参照 **修改DVMT和CFG LOCK** 操作，没解锁CFG Lock 请自行开启 `AppleXcpmCfgLock` 没修改 `DVMT Pre-Allcated` 请使用原版 WhateverGreen.kext 并在 Config.plist 添加核显设备属性 `framebuffer-patch-enable Data 0100000 和 framebuffer-stolenmem Data 00003001`

* **SMBIOS** 请自行补全`Config`默认设定为`UpdateSMBIOSModel:Custom + CustonSmbiosGuid`可以防止 OC 引导其他操作系统注入`PlatformInfo`但是会导致无法安装`Win版BootCamp`控制面板，可以安装前改回`UpdateSMBIOSModel:Overwrite 并关闭 CustonSmbiosGuid`安装后再打开。

* **SystemUUID** 可以选择`Windwos CMD`输入`wmic csproduct get uuid` 获取主板`DmiGuid`填入`SystemUUID`

* **ACPI修补部分** SSDT.aml 包含 PMCR (用于修复电源键不可用的问题) Plugin-type 1（用于加载X86PP不多解释）和屏蔽对于macOS不需要的ACPI、 PNP 和 没有物理设备或物理设备默认禁用的PCI设备,  屏蔽 HPET(高精度计时器) 释放中断资源解决无法加载AppleHDA的问题 **在 Intel 6代及以上芯片组上 HPET 已是过时设备 不再需要HPET提供中断，macOS下启用HPET可能会导致中断冲突问题，在2016年或者更新的Intel Mac电脑上 DSDT 中 HPET 的 `_STA` 判断为 `If (OSDW) {Return (0)}` macOS下屏蔽HPET，而在这些新硬件平台安装 Windows 10 会发现设备管理器 /高精度事件计时器/属性/ 提示该设备未安装驱动，驱动状态：该硬件不需要也无需安装驱动** 

* **声卡驱动方面** 关于DP音频随机失效问题，EFI使用`FakePCIID Match 06C8 Fake A348 (300 Series Q370 原生ID)` AppleALC自带的针对`Comet-Lake-PCH-V (0x6C8)`芯片组`Patch`不是常规的`Device-id Patch`对于显示器音频和板载声卡共用同一个音频控制器的设备(直白说就是 AppleHDADriver 和 AppleHDAHDMI_DPDriver 都在 HDEF@1F,3 下)，虽然可以驱动板载声卡但却因为无法加载`IOClass: AppleHDAHDMI_DPDriver`导致 DP_HDMI_Audio 无法使用，故使用 FakePCIID 仿冒成原生ID来稳定的驱动核心显卡数字音频，
![DP音频输出](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/DPAudio.png)
![IOreg](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/ioreg.png)

**EFI 使用时出现其他问题，或者有其他有价值的意见和建议请提交Issues探讨**

## 注意⚠️ ##
* AppleALC自带的针对 `Device 1736 400 Series PCH HD Audio Patch` 不属于 Device ID 查找替换补丁，而是屏蔽AppleHDAController对音频控制器的硬件ID匹配 会导致`FakePCIID`注入失效请使用EFI中自带的 AppleALC(长期维持更新)

## BIOS 设置
* System Configuration → SATA Operation: ***AHCI***
* Secure Boot → Secure Boot Enable: ***Disabled***
* System Configuration →  SerialPort ***Disabled*** （建议)
* Secure → PTT Secure(TPM) ***Disabled*** （建议)
* Secure → Intel SGX ***Disabled*** （建议)
* System Configuration → PCI Slot ***Disabled*** (如果PCI插槽为空，仅TM系列)

## 关于无法重新启动
* Dell 7080MT使用OC引导时出现了重启系统已经关闭但是主机无法重启的问题 长时间等待还会导致CMOS被自动清理BIOS设置丢失的问题，而使用CLOVER没有此问题，后经查找发现我的 Config.plist ResetAddress 和 ResetValue 填写的 0x64 0xFE 再在MaciASL查看 System FACP 发现对应位置已被更改，关闭补丁后也出现和OC引导同样的问题，由此可看出问题就在这，后经比较发现Dell HP Lenovo等品牌机FACP的ResetAddress都为0xB2 但是ResetValue不唯一 ，Dell 几乎所有机器ResetValue都是0x73 而7080MT之外的机器并没有重启问题，实际 0xB2 I/O是可由OEM制造商分配的端口，自然也没有统一规范 至于7080MT为何无法使用 B2 73 尚不清楚，
* Clover Configurator的注释中找到相关说明如下：
![IOreg](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/RESET.png)
* 关于 OC 的 FadtEnableReset 无效的问题，查看源码发现这个选项仅针对 FACP 中没有声明 ResetAddress 和 ResetValue的机器很显然 Dell 明确为 0xB2 0x73 这个选项没用。。。。
* 在OC的ACPI补丁部分将 FACP的 B2 73 改为 CF9 06或者64 FE 都可以解决重启的问题 

## 关于如何修复Windows 热重启到 macOS 麦克风不可用问题
* Windows到macOS热重启出现部分硬件无法工作的问题已经是老生常谈了，目前也没有什么好的解决办法，还是推荐关机再开机，但是在 0xCF9 I/O端口写入ResetValue不止0x06 根据Intel DataSheet 说明 写入0x02 为 CpuReset 写入 0x06 为 HardReset (也是最常用的重启) ，写入0x0E为 Full Reset (关机再自动开机) 将 FACP的 ResetValue 通过ACPI Patch修改为 0E 即可解决该问题 
* 请自行修改 Config.plist 如果你不用OC引导Windows 请忽略此部分说明

## 修改DVMT和CFG LOCK

* 无法使用`Grub Setup_var` 需要用到Ru.efi (Ru.efi 在/EFI/OC/Tools里) 请将Ru.efi在 BIOS 中添加进 Boot Menus 后启动 进入 Ru 后按 "Alt" + "=" 并查找 **CPUSetup** 和 **SaSetup**
* 解锁"CFG-LOCK" 找到CPUSetup 将 "0030" "0E" 位改为 00 按 `Ctrl + W `保存
![Unlock CFGLOCK.png](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/CFG-LOCK.png)
* 修改DVMT 搜索 SaSetup 将 "00F0" "05" 位改为 "02" 按` Ctrl + W` 保存
![Set 64MB DVMT.png](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/DVMT.png)
