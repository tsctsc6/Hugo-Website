+++
date = '2026-07-18T11:13:03+08:00'
lastmod = '2026-07-18T11:13:03+08:00'
draft = false
title = 'CachyOS 常用命令'
categories = ['Main Sections']
tags = ['Linux']
+++

## 安装新系统

### 安装基本系统

CachyOS 会自带一个小型的操作系统。

进入界面后，会有一个 "CachyOS Hello" 的窗口，上面有一个“启动安装程序”的按钮。

但是先不要急着按。先打开终端 (Konsole) ，自动筛选并更新中国区镜像源：

```bash
# 测速并生成 Arch Linux 镜像源
rate-mirrors arch | sudo tee /etc/pacman.d/mirrorlist

# 测速并生成 CachyOS 专属的优化镜像源
rate-mirrors cachyos | sudo tee /etc/pacman.d/cachyos-mirrorlist
```

> [!info]
> `tee` 命令，它允许用户将标准输入的数据同时写入到标准输出（通常是终端）和一个或多个文件中。

> [!warning]
> 不推荐在 mirrorlist 中只填写一个镜像源。

然后点击“启动安装程序”，根据自己的喜好，选择一些选项，开始安装。

> [!info]
> 文件系统推荐使用 Btrfs ，因为它能为磁盘创建快照，方便在做出错误操作的时候回滚。

安装完毕后，在重启之前，把安装介质（U盘，光盘等）移除，重启。

> [!info]
> 有时候电脑主板可能会开启 Secure Boot ，这样无法进入 CachyOS ，需要在主板的设置中关闭。之后，若是需要，可以为 CachyOS 启用 Secure Boot 。

### 安装基本软件

安装 yay ，它是一个非常流行且强大的 AUR (Arch User Repository) 助手（AUR Helper）和 pacman 包管理器包装器。

```bash
sudo pacman -S yay
```

---

安装输入法，这里我选择 fcitx5-rime + 雾凇拼音(rime-ice)。

```bash
sudo pacman -S fcitx5 fcitx5-configtool fcitx5-qt fcitx5-gtk fcitx5-rime
```

编辑环境变量：

```bash
sudo nano /etc/environment
```

```bash
XMODIFIERS=@im=fcitx
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=fcitx
```

> [!info]
> 如果使用 KDE Plasma 桌面环境，仅需设置 `XMODIFIERS` 即可。
> {{<externalLinkCard title="Using Fcitx 5 on Wayland - Fcitx" link="https://fcitx-im.org/wiki/Using_Fcitx_5_on_Wayland#KDE_Plasma" cover="auto">}}

下载雾凇拼音：

{{<externalLinkCard title="iDvel/rime-ice: Rime 配置：雾凇拼音 | 长期维护的简体词库" link="https://github.com/iDvel/rime-ice" cover="auto">}}

在 Release 中下载 full.zip 。

打开路径 ~/.local/share/fcitx5/rime ，把 full.zip 全部内容解压到这个文件夹中，不用再添加单独的文件夹。

打开设置 -> 键盘 -> 虚拟键盘 -> 选择 fcitx5 。打开设置 -> 输入法 -> 添加“中州韵”

---

安装谷歌浏览器：

```bash
yay -S google-chrome
```

顺便一提，要想谷歌浏览器通过代理，需要添加命令行参数：

```bash
google-chrome-stable --proxy-server="socks5:127.0.0.1:10808"
```

---

Linux 的触控板的滚动操作，是跟 Windows 相反的。也就是说，在触控板双指向上滑动，界面会向上滚动。可以在设置 -> 鼠标和触控板 -> 触控板，勾选“反向滚动”以获得和 Windows 相同的逻辑。

### Btrfs 快照恢复指南

CachyOS 默认配置了 Snapper 。在每次使用 pacman 或 yay 安装、更新、卸载软件之前和大更新时，系统会自动创建一个更新前（Pre）和更新后（Post）的快照。

默认情况下，CachyOS 将 /home（个人文档、下载、游戏等）划分在独立的 Btrfs 子卷中。这意味着系统快照只备份和恢复系统文件（如 /usr, /etc 等）。即使把系统回滚到三天前，昨天下载在主目录的文件和浏览器浏览记录也不会丢失。

Snapper 拥有自动清理机制，会自动删除很久以前的旧快照，只保留最近的若干个。

#### GUI 操作

##### 假如能进入系统

- 在应用程序菜单中搜索并打开 Btrfs Assistant（需要输入管理员密码）。
- 切换到 Snapper 标签页。
- 在 Browse/Restore 子标签页，有一个快照列表，包含了快照的 ID、时间、以及触发原因（比如 pacman 安装软件）。
- 找到想回滚到的那个时间点（通常是出问题前的那个 Pre 快照），选中它。
- 点击 Restore 。
- 重启电脑，系统就会完全回到那个时间点的状态。

##### 假如不能进入系统

- 重启电脑，在看到 CachyOS 的 GRUB 引导菜单（就是选择启动系统的那个界面）时，按下方向键停止倒计时。
- 选中 CachyOS -> Snapshots 并回车。
- 界面会列出历史所有的快照版本，按时间排序。选择出事之前的那个正常版本并回车。
- 系统会以只读模式顺利进入选择的那个历史系统桌面。

> [!Warning]
> 此时只是“穿越”回了过去，但这个过去的系统目前是只读的，还没有真正替换掉当前系统，还需要多做一步操作来真正覆写系统。

- 在应用程序菜单中搜索并打开 Btrfs Assistant（需要输入管理员密码）。
- 切换到 Snapper 标签页。
- 在 Browse/Restore 子标签页。
- 找到想回滚到的那个时间点（通常是出问题前的那个 Pre 快照），选中它。
- 点击 Restore 。
- 重启电脑，系统就会完全回到那个时间点的状态。

##### 手动备份

- 在应用程序菜单中搜索并打开 Btrfs Assistant（需要输入管理员密码）。
- 切换到 Snapper 标签页。
- 在 New/Delete 子标签页，有一个快照列表，包含了快照的 ID、时间、以及触发原因（比如 pacman 安装软件）。
- 点击 New ，输入 Description ，确定。

#### 命令行操作

查看现有快照列表：

```bash
sudo snapper -c root list
```

回滚并重启计算机：

```bash
sudo snapper -c root rollback <id>
sudo reboot
```

创建快照：

```bash
sudo snapper -c root create -d "快照描述"
```

## 游戏

打开 CachyOS Hello -> 应用/调整 -> 安装游戏软件包

系统会自动处理一系列依赖，并且自动安装 Steam 。

如果有其他不在 Steam 的游戏想玩，有这么一种技术方案。

安装 ProtronPlus :

```bash
yay -S ProtonPlus
```

打开 ProtonPlus ，选择展开 DW Proton 并安装最新版本。

重启 Steam 使更改生效。

在 Steam 库中：

- 添加非 Steam 游戏
- 浏览
- 选择 exe 文件。

（可选）：修改 Steam 库的游戏名字。在属性中直接修改名字即可。

- 在属性 -> 兼容性选项卡中，勾选“强制使用特定的...”
- 选择 DW Proton ，然后关闭窗口。

在 Steam 中启动。

如果刚刚选择的 exe 是安装程序，在安装完毕后，需要更改 Steam 的启动目标。

- 打开属性
- 更改目标，使其指向实际安装好的程序。另外，确保路径用引号括起来。

如何找到这个程序？要弄清楚这个问题，需要讲讲这套技术方案的原理。

- Proton（基于 Wine 发展而来）是一个兼容层（Compatibility Layer）。它的原理是实时翻译，把 exe 的指令实时翻译为 Linux 的指令。
- Proton 有 Wine Prefix 机制。 Steam 会在 `~/.steam/steam/steamapps/compatdata/[contener-id]/pfx/` 创建一个虚拟的 C 盘( `drive_c` 文件夹) 。里面的文件和 Windows 的 C 盘基本相同。

所以，刚刚安装好的程序，就在 `~/.steam/steam/steamapps/compatdata/[contener-id]/pfx/drive_c` 之中。

> [!tip]
> 理论上，可以把许多 exe 放到同一个 Wine Prefix 中。要切换启动的应用，就像上文那样，切换目标程序即可。

> [!warning]
> 有些游戏启动器，直接调用其他 exe ，很可能会启动失败。建议直接启动目标 exe 。

## 安装 deb 包

有时某些软件只打包了 deb 包，并且社区没有为其重新打包适配 pacman 包管理器，这时就需要把 deb 包转换为 .pkg.tar.xz 文件。

安装 debtap :

```bash
yay -S debtap
```

更新 debtap 数据库（这个数据库就是把 deb 的依赖映射到 pacman 的依赖的数据库）：

```bash
sudo debtap -u
```

使用 debtap 转换 deb 包

```bash
debtap xxx.deb
```

安装

```bash
sudo pacman -U xxx.pkg.tar.xz
```
