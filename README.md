# 玩客云 OneCloud · ArchLinuxARM 固件（GitHub Actions 云编译）

为玩客云（Amlogic S805 / meson8b，AArch64）构建的 ArchLinuxARM 固件。内核、u-boot 与设备树使用
[hzyitc](https://github.com/hzyitc) 的玩客云适配分支，根文件系统替换为官方 ArchLinuxARM aarch64 基础包，
通过 GitHub Actions 自动编译，并把成品固件发布到 GitHub Releases。

## 产物说明

每次流水线在 Release 中附带以下文件：

| 文件 | 说明 |
| --- | --- |
| `*.img.gz` | SD / U 盘启动镜像，写入存储卡后从外部存储引导 |
| `*.burn.img` | eMMC 线刷包，使用 Amlogic USB Burning Tool 烧录到内置存储 |
| `SHA256SUMS.txt` | 全部固件的 SHA256 校验值 |
| `RELEASE_NOTES.md` | 本次构建的说明文档（版本信息、校验值、刷机步骤） |

## 目录结构

```
.
├── .github/
│   └── workflows/
│       └── build-onecloud-arch.yml   # 云编译 + 自动发布 Release 工作流
├── README.md                         # 本文件
└── FIRMWARE_GUIDE.md                 # 刷机与系统使用详细指南
```

## 触发编译

1. Fork 或新建仓库，把 `.github/workflows/build-onecloud-arch.yml` 放入对应路径。
2. 仓库 **Settings → Actions → General → Workflow permissions** 选择 **Read and write permissions**
   （发布 Release 需要写权限）。
3. 打开 **Actions** 页面，选择 `Build OneCloud ArchLinux (SD + Burn 线刷包)`，点击 **Run workflow**：
   - `RELEASE`：Armbian 基础版本（仅作编译框架，rootfs 最终替换为 Arch），默认 `trixie`。
   - `BUILD_MINIMAL`：最小化内核，推荐 `yes`。
   - `PRERELEASE`：是否标记为预发布。
4. 编译约 40～90 分钟，完成后在仓库 **Releases** 页面下载固件。

> GitHub Runner 默认磁盘较小，工作流第一步已自动扩容并分配 4G swap；如编译中途磁盘不足，
> 可调大 `root-reserve-mb` 与 `swap-size-mb`。

## 刷机方式

### 方式一：SD / U 盘启动（推荐先验证）

1. 从 Release 下载 `*.img.gz` 并解压。
2. 用 [balenaEtcher](https://balena.io/etcher) 或 Rufus 将镜像写入 SD 卡 / U 盘。
3. 插入玩客云，上电引导。

### 方式二：eMMC 线刷（burn.img）

1. 安装 Amlogic USB Burning Tool **v2.1.3**（新版工具容易烧录失败）。
2. 导入 `*.burn.img`，勾选擦除 flash、擦除 bootloader。
3. 双公 USB 线连接电脑与玩客云，使设备进入升级模式。
4. 等待烧录完成，拔线后上电。

## 首次登录

- 默认用户：`root`，默认密码：空（直接回车）。
- 插网线后在路由器后台查看 IP，或用 `arp-scan` 扫描局域网。
- 登录后立即执行 `passwd root` 设置密码。
- SSH 默认开启；串口调试参数：`ttyAML0,115200n8`。

## 常用命令

```bash
pacman -Syu                 # 系统更新
pacman -S docker samba      # 安装软件包
systemctl status sshd       # 查看 SSH 状态
uname -a                    # 查看内核版本
```

## 注意事项

- S805 为 AArch64 架构，请勿混入 armv7 软件包。
- 内核与 u-boot 依赖 hzyitc 适配补丁，主线内核 / 通用 u-boot 无法正常驱动网卡、USB 与 eMMC。
- ArchLinuxARM 基础包不含 meson 固件，已由编译框架从适配分支带入。
- SD 镜像根分区为 `mmcblk0p2`，eMMC 线刷包为 `mmcblk1p2`，fstab 已分别处理，请勿互换镜像。

## 致谢

- [hzyitc/u-boot-onecloud](https://github.com/hzyitc/u-boot-onecloud)
- [hzyitc/linux-meson](https://github.com/hzyitc/linux-meson)
- [Armbian build framework](https://github.com/armbian/build)
- [ArchLinuxARM](https://archlinuxarm.org/)

> （注：部分内容由豆包 AI 生成）