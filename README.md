# Hackintosh-OptiPlex-7080-MT

# 简介

**Dell OptiPlex 7080MT EFI**， 支持**macOS Catalina 10.15.5** 或更高版本

# 硬件配置

* **CPU**: Intel® Core™ i5-10500
* **IGPU**: Intel® UHD Graphics 630
* **RAM**: 8GB DDR4 2666 Daul Channel
* **SSD**: APPLESSD SM0128L Media & Samsung SSD 860 EVO 
* **Wi-Fi & Bluetooth**: BCM943602CDP

# 可用

* CPU 睿频
* IGPU 图形加速，HEVC H.264 En/Decode 
* 仿冒声卡仅针对7080MT的ALC256 LayoutID 67 线路输出独立，可选内置扬声器与耳机 或 线路输出与耳机自动切换 (AppleALC 1.5.6已合并Commit "Add ALC256 layout-id 67 for Dell OptiPlex 7080")
* 麦克风工作需要VerbStub和ComboJack

![演示](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/演示.mov)

* 除最靠近网口的USB2.0(为仅保留15端口而屏蔽)其他的正常
* 以太网卡 
* 睡眠唤醒
* 休眠唤醒 (需要打开 DiscardHibernateMap，可选放入 HibernateFixup.kext，主硬盘SATA则需要开启ThirdPartyDrives，Config→HibernateMode：Auto 系统休眠模式选择25 并加入 Disable RTC Wake Schedule 补丁)

# 补充及说明

## 蓝牙

* 不同的网卡的安装方式可能蓝牙HCI位置可能不同，我的蓝牙USB跳线直接插在主机后的第二个`USB2.0 Type-A` 端口 (`HS09`) 

## 传感器方面🌡️

* 相关驱动请自行添加

## WhateverGreen 
 
* EFI中 Weg 仅包含开机第二阶段花屏补丁、`GFX0 → IGPU`、`HECI → IMEI`的重命名、FakeID
* 有独立显卡的请自行更换原版

## ComboJack_Reloader🤖️

* 自动化程序用来解决插入耳机弹窗忘记选择耳机类型或选错时无需重新插拔耳机直接运行该自动化脚本便可重新调用插孔侦测供您选择

![CRL](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/CRL.mov)

## IOKitPersonalitiesInjector.kext (空壳驱动)
 
* 包含 iMac19,2 SMBIOS 仅核显的`AGPM`注入来修复核显视频处理时满载满频且不保留基本图形渲染性能而导致UI卡顿的问题、
* 修改系统报告 SATA 控制器的名称 GenericAHCI → Intel 14 Series Chipset (仅作修饰)，
* USB电流属性注入 (仅针对11.0 的新`IOProviderClass：AppleUSBHostResources`，之前版本为`AppleBusPowerController`)，可为 iOS 设备提供12W的充电功率 (5v 2.4A)

## DVMT预分配和CFGLock
 
* EFI中不包含DVMT补丁请自行使用 `Ru.efi` 参照 **修改DVMT和CFG LOCK** 操作，
* 没解锁CFG Lock 请自行开启 `AppleXcpmCfgLock` 
* 没修改 `DVMT Pre-Allcated` 请使用原版 WhateverGreen.kext 并在 Config.plist 添加核显设备属性 `framebuffer-patch-enable Data 0100000 和 framebuffer-stolenmem Data 00003001`

## SMBIOS

* 请自行补全`Config`默认设定为`UpdateSMBIOSModel:Custom + CustonSmbiosGuid`可以防止 OC 引导其他操作系统注入`PlatformInfo`但是会导致无法安装`Win版BootCamp`控制面板，可以安装前改回`UpdateSMBIOSModel:Overwrite 并关闭 CustonSmbiosGuid`安装后再打开。

## SystemUUID
 
* 可以选择`Windwos CMD`输入`wmic csproduct get uuid` 获取主板`DmiGuid`填入`SystemUUID`

## ACPI修补部分

* 屏蔽 HPET(高精度计时器) 释放中断资源解决无法加载AppleHDA的问题
* SSDT.aml 包含 PMCR (用于修复电源键不可用的问题) Plugin-type 1（用于加载X86PP不多解释）和屏蔽对于macOS不需要的ACPI、 PNP 和 没有物理设备或物理设备默认禁用的PCI设备

## 声卡驱动方面🔊

* 关于DP音频随机失效问题，EFI使用`FakePCIID Match 06C8 Fake A348 (300 Series Q370 原生ID)` AppleALC自带的针对`Comet-Lake-PCH-V (0x6C8)`芯片组`Patch`不是常规的`Device-id Patch`对于显示器音频和板载声卡共用同一个音频控制器的设备(直白说就是 AppleHDADriver 和 AppleHDAHDMI_DPDriver 都在 HDEF@1F,3 下)，虽然可以驱动板载声卡但却因为无法加载`IOClass: AppleHDAHDMI_DPDriver`导致 DP_HDMI_Audio 无法使用，故使用 FakePCIID 仿冒成原生ID来稳定的驱动核心显卡数字音频，
![DP音频输出](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/DPAudio.png)
![IOreg](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/ioreg.png)

## EFI 使用时出现其他问题，或者有其他有价值的意见和建议请提交Issues探讨🤔️

# 注意⚠️

* AppleALC自带的针对 `Device 1736 400 Series PCH HD Audio Patch` 不属于 Device ID 查找替换补丁，而是屏蔽AppleHDAController对音频控制器的硬件ID匹配 会导致`FakePCIID`注入失效请使用EFI中自带的 AppleALC(长期维持更新)

# BIOS 设置

* System Configuration → SATA Operation: ***AHCI***
* Secure Boot → Secure Boot Enable: ***Disabled***
* System Configuration →  SerialPort ***Disabled*** （建议)
* Secure → PTT Secure(TPM) ***Disabled*** （建议)
* Secure → Intel SGX ***Disabled*** （建议)
* System Configuration → PCI Slot ***Disabled*** (如果PCI插槽为空，仅TM系列)
* VT- Direct I/O **Disabled** (开启可能导致唤醒NVMe读取掉速，不在BIOS关闭也可以☑️Config→Kernel→ Quirks→DisableIoMapper)

# 关于无法重新启动🔄

* Dell 7080MT使用OC引导时出现了重启系统已经关闭但是主机无法重启的问题 长时间等待还会导致CMOS被自动清理BIOS设置丢失的问题，而使用CLOVER没有此问题，后经查找发现我的 Config.plist ResetAddress 和 ResetValue 填写的 0x64 0xFE 再在MaciASL查看 System FACP 发现对应位置已被更改，关闭补丁后也出现和OC引导同样的问题，由此可看出问题就在这，后经比较发现Dell HP Lenovo等品牌机FACP的ResetAddress都为0xB2 但是ResetValue不唯一 ，Dell 几乎所有机器ResetValue都是0x73 而7080MT之外的机器并没有重启问题，实际 0xB2 I/O 是 SMI Command Port，即使macOS下向该端口写入 0x73 也依旧可以重启，但是在 FACP 中不知为何没效果 ASL测试代码如下：
```Swift
Scope (\)
{
    OperationRegion (TEST, SystemIO, 0xB2, One) // 另外测试起始偏移量 0x64 _INI 赋值 TEST = 0xFE 也正常重启
    Field (TEST, ByteAcc, NoLock, Preserve)
    {
        REST,   8
    }
    Method (_INI)
    {
        REST = 0x73 // 当macOS启动执行ACPI后会直接黑屏重启
    }
}
```

* Clover Configurator的注释中找到相关说明如下：
![IOreg](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/RESET.png)
* 关于 OC 的 FadtEnableReset 无效的问题，查看源码发现这个选项仅针对 FACP 中没有声明 ResetAddress 和 ResetValue的机器很显然 Dell 明确为 0xB2 0x73 这个选项没用。。。。
* 在OC的ACPI补丁部分将 FACP的 0xB2 0x73 改为 0xCF9 0x06或者 0x64 0xFE 都可以解决重启的问题 

# 关于如何修复Windows 热重启到 macOS 麦克风不可用问题🎤

* Windows到macOS热重启出现部分硬件无法工作的问题已经是老生常谈了，目前也没有什么好的解决办法，还是推荐关机再开机，但是在 0xCF9 I/O端口写入ResetValue不止0x06 根据Intel DataSheet 说明 写入0x02 为 CpuReset 写入 0x06 为 HardReset (也是最常用的重启) ，写入0x0E为 Full Reset (关机再自动开机) 将 FACP的 ResetValue 通过ACPI Patch修改为 0E 即可解决该问题 
* 请自行修改 Config.plist 如果你不用OC引导Windows 请忽略此部分说明

# 关于如何解决可能出现的唤醒AirDrop自动关闭问题🤦🏻‍♂️

* 通常使用PCIe 1x Wi-Fi转接卡的用户可能普遍存在唤醒AirDrop出于关闭状态的问题，解决方案为重新启动 " Sharingd " 进程，可使用SleepWatcher 进程守护程序 并在 rc.wake 写入 killall sharingd 程序监测到系统唤醒执行从而解决唤醒 AirDrop关闭问题
* 也可在rc.wake 写入 pmset schedule cancelall 达到每次唤醒都清空唤醒计划的目的 (非 PowerNap Wake Schedule) ，或其他命令
* 下载链接 ![点这里下载 Sleepwatcher](https://www.bernhard-baehr.de/sleepwatcher_2.2.1.tgz)

# 可能在7080MT上普遍存在的问题

* 7080MT在BIOS中禁用**Deep Sleep** 开启 **WOL** 可能导致睡眠期间系统自动触发DarkWake (RTC (Alarm) 或者 S3 WOL) 后再次自动睡眠时出现系统已经睡眠（呼吸灯闪烁) 但是主机后的电源的指示灯依旧亮着，且此时无法将电脑从睡眠唤醒，稍等片刻自动关机的问题。

# 修改DVMT和CFG LOCK

* 无法使用`Grub Setup_var` 需要用到Ru.efi (Ru.efi 在/EFI/OC/Tools里) 请将Ru.efi在 BIOS 中添加进 Boot Menus 后启动 进入 Ru 后按 "Alt" + "=" 并查找 **CPUSetup** 和 **SaSetup**
* 解锁"CFG-LOCK" 找到CPUSetup 将 "0030" "0E" 位改为 00 按 `Ctrl + W `保存
![Unlock CFGLOCK.png](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/CFG-LOCK.png)
* 修改DVMT 搜索 SaSetup 将 "00F0" "05" 位改为 "02" 按` Ctrl + W` 保存
![Set 64MB DVMT.png](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/DVMT.png)
