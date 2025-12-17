# 安卓Walkman NW-A100系列设备技术文档

## 操作指南

### <ins>免责声明</ins>
本指南仅适用于Windows设备，仅提供给已掌握基本终端命令、adb或fastboot的高级用户使用。需预先安装ADB工具。

若操作导致设备变砖/损坏，您将自行承担一切后果，本人概不负责且不接受任何投诉。本指南不提供终端用户技术支持，不教授adb安装或基础命令执行等入门知识。

### 前期步骤

请先完成以下操作：

1. 在Walkman开发者选项中启用USB调试
2. 将Walkman连接至Windows电脑
3. 验证adb可用性，执行`adb shell getprop ro.boot.slot_suffix`
4. 记录输出结果（应为"_a"或"_b"）
5. 前往[releases页面](https://github.com/Sikz1218/unlock-and-root_android_walkman_A100-series/releases/tag/v0.0.1)下载相关文件
6. 将所有文件放入工作目录，执行`uuu`验证工具可用性

命令输出结果将作为执行`uuu`命令时的分区后缀参考

### Bootloader解锁

这将清除所有用户数据。
1. 在开发者选项中，启用 OEM 解锁和 ADB 调试。
2. 运行 `adb reboot bootloader` 进入 fastboot 模式。
3. 在 fastboot 模式（SONY 标志）下，运行：
- Mac/Linux: `fastboot oem unlock`
- Windows: `uuu FB: oem unlock`
4. 运行后，界面可能看似卡住，但设备实际上正在尝试擦除 userdata 分区。大约需要 500 秒。
5. 过程完成后，运行以下命令重启设备。
- Mac/Linux: `fastboot reboot`
- Windows: `uuu FB: reboot`

### 禁用AVB

刷写内核需执行此步骤。通过以下命令刷写空白vbmeta文件禁用AVB：
- (若当前槽位为_a): `uuu FB: flash vbmeta_a blank_vbmeta.img`
- (若当前槽位为_b): `uuu FB: flash vbmeta_b blank_vbmeta.img`

首次启动将进入bootloop，随后进入恢复模式提示系统启动失败。此时需选择恢复出厂设置选项（使用音量键导航，电源键确认）。重置后系统应正常启动

### ROOT

修补法：apatch.apk并使用apatch修补boot.img，将修补好的boot文件移动至工作目录，进入fastboot后执行：
- (若当前槽位为_a): `uuu FB: flash boot_a 你修补的boot文件名.img`
- (若当前槽位为_b): `uuu FB: flash boot_b 你修补的boot文件名.img`

内核法：刷入本人编译的KernelSU内核，安装ksu.apk即可获取ROOT权限

### 内核
这个仓库中的内核源码已打补丁，支持 KernelSU、降低 CPU 频率以及更省电的 CPU 频率调控。使用提供的 `walkman.config` 文件作为配置。

我已经预编译好的版本在[这里]()。刷机步骤：进入 fastboot 模式，然后执行：（如果你使用的是 ZX500，请更改文件名）

- Mac/Linux: `fastboot flash boot boot-a100.img`.
- Windows(_a): `uuu FB: flash boot_a boot-a100.img`
- Windows(_b): `uuu FB: flash boot_b boot-a100.img`


## 技术发现

### 解压更新文件

首先，你需要获取设备的密钥字符串。启用 adb，然后执行 `adb shell cat /vendor/usr/data/icx_nvp.cfg`。你可以在 NAS 部分找到你的密钥字符串。通过执行 `java -version` 确保你的路径中有版本大于 1.8 的 java。下载固件解密器 [这里](https://github.com/notcbw/2019_android_walkman/releases/download/v0/nwwmdecrypt.jar)。在终端/CMD/PowerShell 中通过执行 `java -jar nwwmdecrypt.jar -i <输入文件> -o <输出文件> -k <密钥字符串>` 来运行解密器。

解密后，解压 zip 文件。使用 [payload_dumper](https://github.com/vm03/payload_dumper) 解包解压后 zip 文件中的 payload.bin 文件。

### 已解包的 Fastboot 固件

- NW-ZX500 系列 4.06 国际版：[https://drive.google.com/file/d/1TUFwOOrex2miPd41UAhe8ioKbxIv4M0R/view?usp=sharing](https://drive.google.com/file/d/1TUFwOOrex2miPd41UAhe8ioKbxIv4M0R/view?usp=sharing)
- NW-ZX500 系列 4.04 中文版（无谷歌服务，更长电池续航）：[https://drive.google.com/file/d/1z8CucsLx0LJ-0HU50QxVYnx8VHVroP7U/view?usp=sharing](https://drive.google.com/file/d/1z8CucsLx0LJ-0HU50QxVYnx8VHVroP7U/view?usp=sharing)
- NW-A100 系列 4.06 国际版：[https://drive.google.com/file/d/1hiNf9VFeh0osPwbGtI2NeH9T5AHZtUJK/view?usp=sharing](https://drive.google.com/file/d/1hiNf9VFeh0osPwbGtI2NeH9T5AHZtUJK/view?usp=sharing)

### 固件更新文件

固件更新文件的前 128 个字节包含文件标识和 SHA-228 摘要。第一个字节是标识 "NWWM"，接下来的 56 个字节是以 ASCII 十六进制数字存储的 SHA-224 摘要。其余部分未知。

加密数据是标准的 Android OTA 更新压缩文件。转换方案为 AES/CBC/PKCS5Padding。

加密密钥以明文存储在 `/vendor/usr/data/icx_nvp.cfg` 文件中，为 48 个字符的 ASCII 文本。前 32 个字节是 AES 密钥，接下来的 16 个字节是初始化向量。NW-A100 系列和 NW-ZX500 系列拥有不同的密钥。

### Fastboot模式进入方式

开机时同时按住音量减键与快进键
若需进入恢复菜单，请连续按压音量加键与电源键2～5次

### NVP系统

所有配置参数、标志位及密钥均以原始字段形式存储于nvp分区。`/vendor/bin`目录下的`nvp`、`nvpflag`、`nvpinfo`、`nvpnode`、`nvpstr`及`nvptest`为nvp操作调试工具：
- `nvp`：十六进制显示nvp分区内容
- `nvpflag`：查看/写入区域标志等参数
- `nvpstr`：控制nvp中的字符串变量
- 其余工具功能尚未明确
