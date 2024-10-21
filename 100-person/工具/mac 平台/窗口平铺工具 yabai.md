---
title: 窗口平铺工具 yabai
date: 2024-09-11T03:05:06+08:00
updated: 2024-09-11T03:54:16+08:00
dg-publish: false
---

- 使用 yabai 进行窗口管理
- 使用 skhd 对快捷键进行托管

要使用这款工具需要先关闭 SIP

## 关闭 SIP

> 如果不想关闭 SIP，可以使用 [aerospace](https://github.com/nikitabobko/AeroSpace)

关机状态下长按开机键，直到出现设置后松开，进入恢复模式

点击 **选项** 继续

---

点菜单栏 > **实用工具** > **启动安全性实用工具**

勾选降低安全性，只勾 **允许用户管理来自被认可开发者的内核扩展** 就行，接着点 **好**

输入电脑密码，点击好

---

接着点击菜单栏 > **实用工具** > **终端**

在终端输入

```bash
csrutil disable
```

输入 `y` 回车

如果需要输入认证用户，直接把需要关闭的输入，然后输入密码

运行完成之后

```bash
csrutil status
```

查看是否是 `disable` 状态

直接点击左上角苹果图标，关机，之后重新进一下恢复模式，确认 SIP 已经关闭

## 安装两个软件

```sh
$ brew install koekeishiya/formulae/yabai
$ brew install koekeishiya/formulae/skhd
```

接着启动服务

```sh
$ skhd --start-service
$ yabai --start-service
```

这个时候会弹出窗口，需要在设置 > 隐私和安全 > 辅助功能中把这两个启动

然后重新运行上面两个启动命令

## 配置文件

直接使用别人的

创建文件

```sh
$ vim ~/.config/yabai/yabairc
```

输入内容

```sh
# default layout (can be bsp, stack or float)
yabai -m config layout bsp

# new window spawns to the right if vertical split, or bottom if horizontal split
yabai -m config window_placement second_child

# padding set to 12px
yabai -m config top_padding 12
yabai -m config bottom_padding 12
yabai -m config left_padding 12
yabai -m config right_padding 12
yabai -m config window_gap 12

# -- mouse settings --

# center mouse on window with focus
yabai -m config mouse_follows_focus on

# modifier for clicking and dragging with mouse
yabai -m config mouse_modifier alt
# set modifier + left-click drag to move window
yabai -m config mouse_action1 move
# set modifier + right-click drag to resize window
yabai -m config mouse_action2 resize

# when window is dropped in center of another window, swap them (on edges it will split it)
yabai -m mouse_drop_action swap

# disable specific apps
yabai -m rule --add app="^System Settings$" manage=off
yabai -m rule --add app="^Calculator$" manage=off
yabai -m rule --add app="^Karabiner-Elements$" manage=off
yabai -m rule --add app="^QuickTime Player$" manage=off
```

接着创建文件

```sh
$ vim ~/.config/skhd/skhdrc
```

输入内容

```sh
# -- Changing Window Focus --

# change window focus within space
alt - j : yabai -m window --focus south
alt - k : yabai -m window --focus north
alt - h : yabai -m window --focus west
alt - l : yabai -m window --focus east

#change focus between external displays (left and right)
alt - s: yabai -m display --focus west
alt - g: yabai -m display --focus east

# -- Modifying the Layout --

# rotate layout clockwise
shift + alt - r : yabai -m space --rotate 270

# flip along y-axis
shift + alt - y : yabai -m space --mirror y-axis

# flip along x-axis
shift + alt - x : yabai -m space --mirror x-axis

# toggle window float
shift + alt - t : yabai -m window --toggle float --grid 4:4:1:1:2:2

# -- Modifying Window Size --

# maximize a window
shift + alt - m : yabai -m window --toggle zoom-fullscreen

# balance out tree of windows (resize to occupy same area)
shift + alt - e : yabai -m space --balance

# -- Moving Windows Around --

# swap windows
shift + alt - j : yabai -m window --swap south
shift + alt - k : yabai -m window --swap north
shift + alt - h : yabai -m window --swap west
shift + alt - l : yabai -m window --swap east

# move window and split
ctrl + alt - j : yabai -m window --warp south
ctrl + alt - k : yabai -m window --warp north
ctrl + alt - h : yabai -m window --warp west
ctrl + alt - l : yabai -m window --warp east

# move window to display left and right
shift + alt - s : yabai -m window --display west; yabai -m display --focus west;
shift + alt - g : yabai -m window --display east; yabai -m display --focus east;

# move window to prev and next space
shift + alt - p : yabai -m window --space prev;
shift + alt - n : yabai -m window --space next;

# move window to space #
shift + alt - 1 : yabai -m window --space 1;
shift + alt - 2 : yabai -m window --space 2;
shift + alt - 3 : yabai -m window --space 3;
shift + alt - 4 : yabai -m window --space 4;
shift + alt - 5 : yabai -m window --space 5;
shift + alt - 6 : yabai -m window --space 6;
shift + alt - 7 : yabai -m window --space 7;

# -- Starting/Stopping/Restarting Yabai --

# stop/start/restart yabai
ctrl + alt - q : brew services stop yabai
ctrl + alt - s : brew services start yabai
ctrl + alt - r : brew services restart yabai
```

快捷键解释：

**改变窗口焦点相关**

- `alt - j`：将窗口焦点移至当前空间内的南边窗口
- `alt - k`：将窗口焦点移至当前空间内的北边窗口
- `alt - h`：将窗口焦点移至当前空间内的西边窗口
- `alt - l`：将窗口焦点移至当前空间内的东边窗口
- `alt - s`：将焦点切换至左边外部显示器
- `alt - g`：将焦点切换至右边外部显示器

**修改布局相关**

- `shift + alt - r`：顺时针旋转布局 270 度
- `shift + alt - y`：沿 y 轴翻转布局
- `shift + alt - x`：沿 x 轴翻转布局
- `shift + alt - t`：切换窗口浮动状态并设置网格为 4:4:1:1:2:2

**修改窗口大小相关**

- `shift + alt - m`：切换窗口的全屏缩放
- `shift + alt - e`：平衡窗口树（使窗口大小占相同区域）

**移动窗口相关**

- `shift + alt - j`：与南边窗口交换位置
- `shift + alt - k`：与北边窗口交换位置
- `shift + alt - h`：与西边窗口交换位置
- `shift + alt - l`：与东边窗口交换位置
- `ctrl + alt - j`：移动窗口至南边并分割
- `ctrl + alt - k`：移动窗口至北边并分割
- `ctrl + alt - h`：移动窗口至西边并分割
- `ctrl + alt - l`：移动窗口至东边并分割
- `shift + alt - s`：将窗口移至左边显示器并将焦点移至左边显示器
- `shift + alt - g`：将窗口移至右边显示器并将焦点移至右边显示器
- `shift + alt - p`：将窗口移至上一个空间
- `shift + alt - n`：将窗口移至下一个空间
- `shift + alt - 1` - `shift + alt - 7`：分别将窗口移至对应 1 - 7 号空间

**启动 / 停止 / 重启 Yabai 相关**

- `ctrl + alt - q`：停止 yabai 服务
- `ctrl + alt - s`：启动 yabai 服务
- `ctrl + alt - r`：重启 yabai 服务

### 按键

魔改过的

```
fn

cmd
lcmd
rcmd

shift
lshift
rshift

alt
lalt
ralt

ctrl
lctrl
rctrl

hyper (cmd + shift + alt + ctrl)

meh (shift + alt + ctrl)
```

其他

```
"return"     (kVK_Return)
"tab"        (kVK_Tab)
"space"      (kVK_Space)
"backspace"  (kVK_Delete)
"escape"     (kVK_Escape)

The following keys can not be used with the fn-modifier:

"delete"     (kVK_ForwardDelete)
"home"       (kVK_Home)
"end"        (kVK_End)
"pageup"     (kVK_PageUp)
"pagedown"   (kVK_PageDown)
"insert"     (kVK_Help)
"left"       (kVK_LeftArrow)
"right"      (kVK_RightArrow)
"up"         (kVK_UpArrow)
"down"       (kVK_DownArrow)
"f1"         (kVK_F1)
"f2"         (kVK_F2)
"f3"         (kVK_F3)
"f4"         (kVK_F4)
"f5"         (kVK_F5)
"f6"         (kVK_F6)
"f7"         (kVK_F7)
"f8"         (kVK_F8)
"f9"         (kVK_F9)
"f10"        (kVK_F10)
"f11"        (kVK_F11)
"f12"        (kVK_F12)
"f13"        (kVK_F13)
"f14"        (kVK_F14)
"f15"        (kVK_F15)
"f16"        (kVK_F16)
"f17"        (kVK_F17)
"f18"        (kVK_F18)
"f19"        (kVK_F19)
"f20"        (kVK_F20)
 
"sound_up"          (NX_KEYTYPE_SOUND_UP)
"sound_down"        (NX_KEYTYPE_SOUND_DOWN)
"mute"              (NX_KEYTYPE_MUTE)
"play"              (NX_KEYTYPE_PLAY)            
"previous"          (NX_KEYTYPE_PREVIOUS)        
"next"              (NX_KEYTYPE_NEXT)
"rewind"            (NX_KEYTYPE_REWIND)          
"fast"              (NX_KEYTYPE_FAST)            
"brightness_up"     (NX_KEYTYPE_BRIGHTNESS_UP)
"brightness_down"   (NX_KEYTYPE_BRIGHTNESS_DOWN)
"illumination_up"   (NX_KEYTYPE_ILLUMINATION_UP) 
"illumination_down" (NX_KEYTYPE_ILLUMINATION_DOWN)
```

按键代码可以通过 `skhd --observe` 进入之后点击按键查看（严格大小写），`Ctrl + c` 退出

## 其他

在 Windows 上，也可以尝试 bug.n, 这是一款基于 [AutoHotKey](https://www.autohotkey.com/) 的窗口管理工具，与 yabai 有着类似的功能：

- [koekeishiya/yabai: A tiling window manager for macOS based on binary space partitioning](https://github.com/koekeishiya/yabai)

如果想了解关于平铺式窗口管理器的更多内容，可以参考如下链接：

- [Why Use A Tiling Window Manager? Speed, Efficiency and Customization! – YouTube](https://www.youtube.com/watch?v=Lj1IfdKY0CU)
- [ChunkWM tutorial on macOS! – YouTube](https://www.youtube.com/watch?v=ZUMucXKU4Fw)
- [CHUNKWM + SKHD – Mac OS X Mojave deserves a Tiling Window Manager system too – YouTube](https://www.youtube.com/watch?v=k1YChPy8_L0)
- [5 reasons the i3 window manager makes Linux better | Opensource.com](https://opensource.com/article/18/8/i3-tiling-window-manager)
- [平铺式窗口管理器 Musca 初体验 · LinuxTOY](https://linuxtoy.org/archives/musca.html)
- [平铺式窗口管理器-Awesome | HaHack](https://www.hahack.com/tools/awesome/)
- [平铺式窗口管理器——Awesome · LinuxTOY](https://linuxtoy.org/archives/awesome.html)

## 卸载

```sh
yabai --stop-service
yabai --uninstall-service
sudo yabai --uninstall-sa
brew uninstall yabai
rm -rf /tmp/yabai_$USER.out.log
rm -rf /tmp/yabai_$USER.err.log
rm ~/.yabairc # 其他配置文件 ~/.config/yabai/yabairc
rm /tmp/yabai_$USER.lock
rm /tmp/yabai_$USER.socket
rm /tmp/yabai-sa_$USER.socket
killall Dock

skhd --stop-service
skhd --uninstall-service
brew uninstall skhd
rm ~/.config/skhd/skhdrc
```
