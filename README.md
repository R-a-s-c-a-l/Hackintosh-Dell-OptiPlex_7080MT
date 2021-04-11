# Hackintosh-OptiPlex-7080-MT

## 简介

**Dell OptiPlex 7080MT EFI**， 支持**macOS Catalina 10.15.5** 或更高版本

### 硬件配置

* **CPU**: Intel® Core™ i5-10500
* **IGPU**: Intel® UHD Graphics 630
* **RAM**: 8GB DDR4 2666 Daul Channel
* **SSD**: APPLESSD SM0128L Media & Samsung SSD 860 EVO 
* **Wi-Fi & Bluetooth**: BCM943602CDP

### 可用

* CPU 睿频
* IGPU 图形加速，HEVC H.264 En/Decode 
* 仿冒声卡仅针对7080MT的ALC256 LayoutID 67 线路输出独立，可选内置扬声器与耳机 或 线路输出与耳机自动切换 (AppleALC 1.5.6已合并Commit "Add ALC256 layout-id 67 for Dell OptiPlex 7080")
* 麦克风工作需要VerbStub和ComboJack

![演示](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/演示.mov)

* 除最靠近网口的USB2.0(为仅保留15端口而屏蔽)其他的正常
* 以太网卡 
* 睡眠唤醒
* 休眠唤醒 (需要打开 DiscardHibernateMap，如果主硬盘是 _SATA_ 则需要开启 _ThirdPartyDrives，Config → HibernateMode：Auto_ 系统休眠模式选择 25 并加入 _Disable RTC Wake Schedule_ 补丁)

## 补充及说明

### 蓝牙

* 不同的网卡的安装方式可能蓝牙HCI位置可能不同，我的蓝牙USB跳线直接插在主机后的第二个 _USB2.0 Type-A_ 端口 (_HS09_) 

### 传感器方面🌡️

* 相关驱动请自行添加

### WhateverGreen 
 
* EFI 中 Weg 仅包含开机第二阶段花屏补丁、 _GFX0 → IGPU HECI → IMEI_ 的重命名、 _FakeID_
* 有独立显卡的请自行更换原版

### ComboJack_Reloader🤖️

* 自动化程序用来解决插入耳机弹窗忘记选择耳机类型或选错时无需重新插拔耳机直接运行该自动化脚本便可重新调用插孔侦测供您选择

![CRL](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/CRL.mov)

### IOKitPersonalitiesInjector.kext (空壳驱动)
 
* 包含 iMac19,2 SMBIOS 仅核显的 _AGPM_ 注入来修复核显视频处理时满载满频且不保留基本图形渲染性能而导致UI卡顿的问题、和 _USB_ 定制

### DVMT预分配和CFGLock
 
* EFI中不包含 _DVMT_ 补丁请自行使用 _Ru.efi_ 参照 **修改DVMT和CFG LOCK** 操作，
* 没解锁 _CFG Lock_ 请自行开启 _AppleXcpmCfgLock_ 
* 没修改 _DVMT Pre-Allcated_ 请使用原版 _WhateverGreen.kext_ 并在 _Config.plist_ 添加核显设备属性:
  > framebuffer-patch-enable Data 0100000 和 framebuffer-stolenmem Data 00003001

### SMBIOS

* 请自行补全 _Config_ 默认设定为 _UpdateSMBIOSModel:Custom + CustonSmbiosGuid_ 可以防止 OC 引导其他操作系统注入 _PlatformInfo_ 但是会导致无法安装 Win 版 BootCamp 控制面板，可以安装前改回 _UpdateSMBIOSModel:Overwrite_ 并关闭 _CustonSmbiosGuid_ 安装后再打开。

### SystemUUID
 
* 可以选择 _Windwos CMD_ 输入 _wmic csproduct get uuid_ 获取主板 _DmiGuid_ 填入 _SystemUUID_

### ACPI修补部分

* 屏蔽 HPET(高精度计时器) 释放中断资源解决无法加载 _AppleHDA_ 的问题
* _SSDT.aml_ 包含 _PMCR_ (用于修复电源键不可用的问题) _Plugin-type 1_ （用于加载X86PP不多解释）和屏蔽对于macOS不需要的 _ACPI、 PNP_ 和 没有物理设备或物理设备默认禁用的PCI设备
* 通过每个 _PCIe Root Port_ 下的 _PRES_ 方法的返回值屏蔽没有安装拓展卡或者出厂被屏蔽的桥 _ACPI_ 设备

### BIOS 设置

* System Configuration → SATA Operation: ***AHCI***
* Secure Boot → Secure Boot Enable: ***Disabled***
* System Configuration →  SerialPort ***Disabled*** （建议)
* Secure → PTT Secure(TPM) ***Disabled*** （建议)
* Secure → Intel SGX ***Disabled*** （建议)
* System Configuration → PCI Slot ***Disabled*** (如果PCI插槽为空，仅TM系列)
* VT- Direct I/O **Disabled** (开启可能导致唤醒NVMe读取掉速，不在BIOS关闭也可以☑️Config→Kernel→ Quirks→DisableIoMapper)

### 关于无法重新启动🔄

* Dell 7080MT使用OC引导时出现了重启系统已经关闭但是主机无法重启的问题 长时间等待还会导致CMOS被自动清理BIOS设置丢失的问题，而使用CLOVER没有此问题，后经查找发现我的 _Config.plist_ _ResetAddress_ 和 _ResetValue_ 填写的 0x64 0xFE 再在 _MaciASL_ 查看 _System FACP_ 发现对应位置已被更改，关闭补丁后也出现和 _OC_ 引导同样的问题，由此可看出问题就在这，后经比较发现 _Dell HP Lenovo_ 等品牌机 _FACP_ 的 _ResetAddress_ 都为0xB2 但是 _ResetValue_ 不唯一 ，Dell 几乎所有机器 _ResetValue_ 都是0x73 而7080MT之外的机器并没有重启问题，实际 0xB2 I/O 是 _SMI Command Port_ ，即使 _macOS_ 下向该端口写入 0x73 也依旧可以重启，但是在 FACP 中不知为何没效果,  ASL测试代码如下：
```Swift
Scope (\)
{
    OperationRegion (TEST, SystemIO, 0xB2, One) // 另外测试起始偏移量 0x64 _INI 赋值 REST = 0xFE 也正常重启
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

* _Clover Configurator_ 的注释中找到相关说明如下：
![IOreg](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/RESET.png)
* 关于 OC 的 FadtEnableReset 无效的问题，查看源码发现这个选项仅针对 FACP 中没有声明 _ResetAddress_ 和 _ResetValue_ 的机器很显然 Dell 明确为 0xB2 0x73 这个选项没用。。。。
* 在OC的ACPI补丁部分将 FACP的 0xB2 0x73 改为 0xCF9 0x06或者 0x64 0xFE 都可以解决重启的问题 

### 关于如何修复Windows 热重启到 macOS 麦克风不可用问题🎤

* Windows到macOS热重启出现部分硬件无法工作的问题已经是老生常谈了，目前也没有什么好的解决办法，还是推荐关机再开机，但是在 0xCF9 I/O端口写入ResetValue不止0x06 根据Intel DataSheet 说明 写入0x02 为 CpuReset 写入 0x06 为 HardReset (也是最常用的重启) ，写入0x0E为 Full Reset (关机再自动开机) 将 FACP的 ResetValue 通过ACPI Patch修改为 0E 即可解决该问题 
* 请自行修改 Config.plist 如果你不用OC引导Windows 请忽略此部分说明

### 关于可能存在的唤醒AirDrop自动关闭的问题😓
* 通常使用PCIe 1x Wi-Fi转接卡的用户可能普遍存在唤醒AirDrop出于关闭状态的问题，解决方案为重新启动 " Sharingd " 进程，可使用SleepWatcher 进程守护程序 并在 rc.wake 写入 killall sharingd 程序监测到系统唤醒执行从而解决唤醒 AirDrop关闭问题
* 也可在rc.wake 写入 pmset schedule cancelall 达到每次唤醒都清空唤醒计划的目的 (非 PowerNap Wake Schedule) ，或其他命令
* 下载链接 [点这里下载 Sleepwatcher](https://www.bernhard-baehr.de/sleepwatcher_2.2.1.tgz)

### 可能在7080MT上普遍存在的问题

* 7080MT在BIOS中禁用**Deep Sleep** 开启 **WOL** 可能导致睡眠期间系统自动触发DarkWake (RTC (Alarm) 或者 S3 WOL) 后再次自动睡眠时出现系统已经睡眠（呼吸灯闪烁) 但是主机后的电源的指示灯依旧亮着，且此时无法将电脑从睡眠唤醒，稍等片刻自动关机的问题。

### 修改DVMT和CFG LOCK

* 无法使用`Grub Setup_var` 需要用到Ru.efi (Ru.efi 在/EFI/OC/Tools里) 请将Ru.efi在 BIOS 中添加进 Boot Menus 后启动 进入 Ru 后按 "Alt" + "=" 并查找 **CPUSetup** 和 **SaSetup**
* 解锁"CFG-LOCK" 找到CPUSetup 将 "0030" "0E" 位改为 00 按 `Ctrl + W `保存
![Unlock CFGLOCK.png](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/CFG-LOCK.png)
* 修改DVMT 搜索 SaSetup 将 "00F0" "05" 位改为 "02" 按` Ctrl + W` 保存
![Set 64MB DVMT.png](https://github.com/R-a-s-c-a-l/Hackintosh-Dell-OptiPlex_7080MT/blob/main/Pic/DVMT.png)

## EFI 使用时出现其他问题，或者有其他有价值的意见和建议请提交Issues探讨🤔️
