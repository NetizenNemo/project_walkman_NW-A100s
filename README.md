索尼A105折腾文档
操作指南
本指南仅适用于已掌握基础终端命令、ADB 或 Fastboot 工具的高级用户。操作前需安装安卓平台工具（包含 ADB 等组件）。
免责声明
以下步骤仅在 Mac 上测试过。若你的设备变砖或损坏，本人不承担任何责任，也不会处理你的抱怨。我不解决你操作端的技术问题，也不会教授 ADB 安装、命令执行等基础操作。
仅适用于 Windows 用户
请先完成以下步骤：
在你的随身听开发者选项中开启 USB 调试
将随身听连接到 Windows 电脑
确保能正常使用adb，执行命令：adb shell getprop ro.boot.slot_suffix
记录输出结果，应为 “_a” 或 “_b”
点击此处下载 uuu 工具
将uuu.exe放入工作目录，执行uuu测试工具是否可用
该命令的输出结果将作为执行uuu命令时的分区后缀。
解锁引导加载程序
此操作会清除所有用户数据：
在开发者选项中，开启 OEM 解锁和 ADB 调试
执行adb reboot bootloader进入 Fastboot 模式
设备显示 SONY 标志（Fastboot 模式）后，执行对应命令：
Mac/Linux：fastboot oem unlock
Windows：uuu FB: oem unlock
执行后设备看似卡住，实际正在擦除用户数据分区，约需 500 秒
操作完成后，执行以下命令重启设备：
Mac/Linux：fastboot reboot
Windows：uuu FB: reboot
关闭 AVB 验证
此步骤是使用自定义内核的必要前提。要关闭 AVB，请执行以下命令刷入空白 vbmeta 文件：
Mac/Linux：fastboot --disable-verity --disable-verification flash vbmeta blank_vbmeta.img
Windows（分区后缀为_a）：uuu FB: flash vbmeta_a blank_vbmeta.img
Windows（分区后缀为_b）：uuu FB: flash vbmeta_b blank_vbmeta.img
设备会先进入开机循环，随后启动到恢复模式并提示安卓系统启动失败。此时需选择 “恢复出厂设置” 选项，用音量键控制光标、电源键确认。恢复出厂设置后系统即可正常启动。
内核操作
本仓库中的内核源码已打补丁，支持 KernelSU、更低 CPU 频率调节及更省电的 CPU 调频策略。请使用提供的walkman.config作为配置文件。
我预编译好的内核可从此处下载。刷入步骤：进入 Fastboot 模式后执行对应命令（若为 A100 机型，修改文件名）：
Mac/Linux：fastboot flash boot boot-zx500.img
Windows（分区后缀为_a）：uuu FB: flash boot_a boot-zx500.img
Windows（分区后缀为_b）：uuu FB: flash boot_b boot-zx500.img
修改区域设置（解除 ZX500 音量限制，A100 机型未测试且可能无效）
完成上述所有操作（解锁引导加载程序、关闭 AVB、刷入提供的内核）
从此处下载并安装 KernelSU 应用
打开 KernelSU 应用，为 shell 开启超级用户权限
在开发者选项中开启 USB 调试
通过 USB 连接随身听与电脑，执行adb shell进入 shell 界面
执行su -获取 root 权限
依次执行nvpflag shp 0x00000006 0x00000000和nvpflag sid 0x00000000，将区域代码切换为 E（适用于阿联酋、东南亚、香港、韩国及大洋洲市场，支持高增益）
断开 USB 连接，重启随身听，高增益选项即可使用
相关发现
解压更新文件
首先获取设备的密钥字符串：开启 ADB 后执行adb shell cat /vendor/usr/data/icx_nvp.cfg，在 NAS 部分找到密钥
执行java -version确保 Java 版本高于 1.8
从此处下载固件解密工具
在终端 / 命令提示符 / PowerShell 中执行java -jar nwwmdecrypt.jar -i <输入文件> -o <输出文件> -k <密钥字符串>解密固件
解密完成后解压压缩包，使用payload_dumper解压包内的 payload.bin 文件。
已解压的 Fastboot 固件
NW-ZX500 系列 4.06 国际版：https://drive.google.com/file/d/1TUFwOOrex2miPd41UAhe8ioKbxIv4M0R/view?usp=sharing
NW-ZX500 系列 4.04 国行版（无谷歌服务，续航更优）：https://drive.google.com/file/d/1z8CucsLx0LJ-0HU50QxVYnx8VHVroP7U/view?usp=sharing
NW-A100 系列 4.06 国际版：https://drive.google.com/file/d/1hiNf9VFeh0osPwbGtI2NeH9T5AHZtUJK/view?usp=sharing
固件更新文件结构
固件更新文件的前 128 字节包含文件标识和 SHA-224 摘要：第 1 段是标识 “NWWM”，接下来 56 字节是 ASCII 十六进制格式的 SHA-224 摘要，其余部分用途未知。
加密数据是标准的安卓 OTA 更新压缩包，加密方式为 AES/CBC/PKCS5Padding。
加密密钥以明文形式存储在/vendor/usr/data/icx_nvp.cfg中，是 48 字符的 ASCII 文本：前 32 字节为 AES 密钥，后 16 字节为初始化向量。NW-A100 和 NW-ZX500 系列的密钥不同。
快捷键进入各类模式
进入 Fastboot 模式：开机时按住音量 + 键和后退键
进入恢复模式：开机时按住音量 - 键和快进键
然后快速按下音量 + 键和电源键 2-5 次，即可进入恢复菜单（特别感谢此指南）
NVP 分区说明
设备的所有配置、标识位、密钥等均以原始字段形式存储在 NVP 分区中。/vendor/bin目录下的nvp、nvpflag、nvpinfo、nvpnode、nvpstr和nvptest被认为是用于操作 NVP 值的调试工具：
nvp：以十六进制格式显示二进制分区内容
nvpflag：查看和写入区域等标识位
nvpstr：控制 NVP 中的其他字符串变量
其余工具的用途未知
