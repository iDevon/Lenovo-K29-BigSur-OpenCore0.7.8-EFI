# Lenovo-K29-BigSur-OpenCore0.7.8-EFI

>设备及功能正常：网卡、无线、蓝牙、触控板、摄像头、读卡器、CPU睿频、隔空投送、合盖睡眠、唤醒、投屏、亮度调整、音量调整
>仅指纹传感器不可用

**K29声卡ALC269 仿冒ID为28**

亮度+`Pause`	亮度-`FN+Delete`	音量+`FN+Right`	音量-`FN+Left`


**关于`Config.plist`配置请参照：**
https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/ivy-bridge.html


**蓝牙睡眠唤醒问题：**

我机型使用的无线网卡：`Broadcom94360HMB` 蓝牙芯片组`20702A3`

将`BrcmFirmwareData.kext`与`BrcmPatchRAM3.kext`安装到`/Library/Extensions`

蓝牙驱动下载：https://github.com/acidanthera/BrcmPatchRAM

其它蓝牙`Intel`及`Atheros`请参照如下链接：

Intel:https://dortania.github.io/Wireless-Buyers-Guide/misc/intel.html

Atheros:https://github.com/zxystd/AthBluetoothFirmware


**风扇转速显示正常**

![截屏2022-03-06 上午11 34 33](https://user-images.githubusercontent.com/86851841/156908108-6bffdabd-d46b-4a33-aae6-2169b5b258f2.png)

**在DSDT中H-EC上面添加SMCD**
```
Device (SMCD)
                {
                    Name (_HID, "FAN00000")  // _HID: Hardware ID
                    Name (_CID, "monitor")  // _CID: Compatible ID
                    Name (TACH, Package (0x02)
                    {
                        "System Fan", 
                        "FAN0"
                    })
                    Method (FAN0, 0, NotSerialized)
                    {
                        Local0 = (^^H_EC.FSP1 << 0x08)
                        Local0 |= ^^H_EC.FSPD /* \_SB_.PCI0.LPCB.H_EC.FSPD */
                        Return (Local0)
                    }
                }
```
**查找**
```
                        Offset (0x95), 
                        FSP1,   8, 
```
**修改为：**
```
                        Offset (0x94), 
                        FSPD,   8, 
                        FSP1,   8, 
```
**并在DeviceProperties下内建通用EC控制器**

参照来源: https://github.com/acidanthera/VirtualSMC/blob/master/Docs/EmbeddedControllers.md#lenovo-g480

添加如下代码：

```
			<key>PciRoot(0x0)/Pci(0x1F,0x0)</key>
			<dict>
				<key>ec-device</key>
				<string>generic</string>
				<key>fan-count</key>
				<integer>1</integer>
				<key>fan0-addr</key>
				<integer>149</integer>
				<key>fan0-big</key>
				<integer>1</integer>
				<key>fan0-div</key>
				<integer>11</integer>
				<key>fan0-inverse</key>
				<integer>1</integer>
				<key>fan0-mul</key>
				<integer>240</integer>
				<key>fan0-size</key>
				<integer>1</integer>
			</dict>
```

如图：

![截屏2022-03-06 上午11 36 38](https://user-images.githubusercontent.com/86851841/156908140-7111beb5-dfd0-43e7-9fbb-0f80ae1e7e2e.png)

用`ssdtPRGen.sh-17.0`生成CPU变频配置文件

CPU为i7-3630QM 非同款CPU请勿加载`ssdt.aml`

CPU开启睿频用`CPUFriend`工具生成`CPUFriendDataProvider.kext`或`SSDT_data.aml`不可同时使用。

**工具下载：**

ssdtPRGen https://github.com/Piker-Alpha/ssdtPRGen.sh

CPUFriend https://github.com/acidanthera/CPUFriend

特别感谢：

[@mjs520](https://github.com/mjs520)提供读取风扇转速的方案https://github.com/acidanthera/VirtualSMC/pull/64
