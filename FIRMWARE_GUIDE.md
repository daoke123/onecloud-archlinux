# 玩客云 ArchLinuxARM 固件 · 刷机与使用指南

## 一、硬件准备

| 物品 | 说明 |
| --- | --- |
| 玩客云设备 | Amlogic S805 / meson8b，AArch64 |
| 电脑 | Windows（线刷工具）或任意可写盘系统（Etcher） |
| 存储卡 / U 盘 | ≥ 4GB，SD 卡建议 Class 10 |
| 双公 USB 线 | 仅 eMMC 线刷需要 |
| 网线 | 必须，固件默认无无线网卡 |
| TTL 串口（可选） | 排针 3.3V TX/RX/GND，115200 波特率，用于排查启动问题 |

## 二、获取固件

在仓库 **Releases** 页面下载最新版本：

- `*.img.xz`：SD / U 盘镜像（需先解压）
- `*.burn.img`：eMMC 线刷包
- `SHA256SUMS.txt`：校验值

下载后先校验完整性：

```bash
# Linux / macOS
sha256sum -c SHA256SUMS.txt

# Windows（PowerShell）
Get-FileHash *.img.xz -Algorithm SHA256
```

## 三、方式一：SD / U 盘启动

1. 解压 `*.img.xz` 得到 `.img` 文件（例如 `xz -d *.img.xz` 或使用 7-Zip）。
2. 打开 [balenaEtcher](https://balena.io/etcher)：
   - Flash from file：选择解压后的 `.img`
   - Select target：选择 SD 卡 / U 盘
   - Flash! 写入并校验。
3. 玩客云断电，插入存储卡 / U 盘，再接电启动。
4. 在路由器后台或局域网扫描中找到设备 IP。

> 从 SD 启动时根分区设备为 `/dev/mmcblk0p2`；线刷到 eMMC 后为 `/dev/mmcblk1p2`。

## 四、方式二：eMMC 线刷（burn.img）

1. 安装 **Amlogic USB Burning Tool v2.1.3**（不建议使用更新版本，部分版本识别设备异常）。
2. 打开工具 → File → Import image，选择 `*.burn.img`。
3. 右键任务 → Erase flash（擦除 flash）、Erase bootloader（擦除引导）按需勾选；全新线刷建议两项都勾。
4. 玩客云断电，双公 USB 线一头插电脑 USB 口，一头插玩客云靠近 HDMI 口的 USB 口。
5. 让设备进入升级模式：短接主板上的升级触点，或按住复位键再接电，工具识别到设备后进度条点亮。
6. 点击 Start，等待烧录完成（绿色勾），拔线、上电。

## 五、首次进入系统

### 1. 登录

```bash
ssh root@<设备IP>
# 密码为空，直接回车
```

### 2. 基础初始化

```bash
# 设置 root 密码
passwd root

# 时区与时间
timedatectl set-timezone Asia/Shanghai
date

# 更新软件源（镜像已预置中科大 ArchLinuxARM 源）
pacman -Syu

# 安装常用工具
pacman -S vim htop usbutils net-tools openssh
```

### 3. 网络查看

```bash
ip addr show eth0
ip route
# 查看已监听端口
ss -tulpn
```

### 4. SSH 配置（可选加固）

```bash
vim /etc/ssh/sshd_config
# PermitRootLogin yes / no；修改后重启
systemctl restart sshd
systemctl enable sshd
```

## 六、常用软件部署示例

### Docker

```bash
pacman -S docker
systemctl enable --now docker
# 免 sudo 使用 docker（可选，新建普通用户后）
usermod -aG docker alarm
```

### Samba 共享

```bash
pacman -S samba
# 编辑 /etc/samba/smb.conf 后设置密码
smbpasswd -a root
systemctl enable --now smb.service nmb.service
```

### Tailscale（内网穿透 / 组网）

```bash
# ArchLinuxARM 需先按官方文档配置 tailscale 源，或使用静态二进制
pacman -S tailscale
tailscale up
```

## 七、根分区扩容

镜像默认 rootfs 较小，卡容量剩余空间需手动扩容：

```bash
# 查看分区
lsblk
# 扩容第二个分区（以 mmcblk0 为例）
parted /dev/mmcblk0
# resizepart 2 100%
resize2fs /dev/mmcblk0p2
```

## 八、故障排查

| 现象 | 排查方向 |
| --- | --- |
| 工具识别不到设备 | 换双公线、换 USB 口、重装 v2.1.3 工具、检查是否进入升级模式 |
| 开机网卡不识别 | 确认使用的是 hzyitc 适配内核；主线内核缺 meson8b 网卡补丁 |
| 串口无输出 | 检查 TX/RX 是否接反、波特率 115200、电平 3.3V |
| 启动 kernel panic | 核对 root= 设备号：SD 为 `mmcblk0p2`，eMMC 为 `mmcblk1p2`；确认未混用 armv7 包 |
| pacman 同步失败 | 检查网络连通性，必要时更换 `/etc/pacman.d/mirrorlist` 中的源 |
| 编译中途磁盘满 | 调大工作流 `root-reserve-mb` 与 `swap-size-mb` |

## 九、救砖

- 能进 USB 升级模式：重新线刷 `burn.img` 即可。
- 完全无响应：TTL 串口观察 u-boot 日志；必要时短接 recovery 触点重新进入线刷模式。
- u-boot 损坏：需通过烧录工具重写引导分区，请勿随意刷入通用 u-boot。

> （注：部分内容由豆包 AI 生成）