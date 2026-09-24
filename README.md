# Keyball61

**简体中文** &nbsp;|&nbsp; [English](#english)

---

## 简体中文

### 关于 Keyball61

Keyball61 是一块 61 键的无线分体键盘：nice!nano 主控、内置轨迹球，屏幕可以选
OLED 或 nice!view，属于 Yowkees 的 keyball 系列；本仓库是它的 ZMK 配置。

感谢：PCB [yangxing844](https://github.com/yangxing844)、外壳
[delock](https://github.com/delock)、固件 [Amos698](https://github.com/Amos698)。

这份 README 在所有分支上内容相同，先介绍键盘本身，再逐分支说明差异。

### 分支一览

本仓库有两条固件主线，分支名基本能读出差异：`nv` = nice!view 屏幕，
`leftball` = 轨迹球挪到左半，`dualball` = 两边都装球，`noball` = 不装球，
`dongle` = 用接收器当主机。

* **基础线**：`zmkfirmware/zmk@v0.3` + tangbonze 版 PMW3610 驱动
  （devicetree 兼容名 `zmk,pmw3610`，CPI、snipe、滚轮层等参数都写在驱动配置里）
* **DYA 线**：cormoran 的 DYA Studio 栈，`cormoran/zmk@v0.3-branch+dya` +
  `cormoran,pmw3610` 驱动 + custom Studio RPC 模块，可以在网页上在线调参

#### 基础线分支

| 分支 | 轨迹球 | 屏幕 | 主手（中央） | 说明 |
| --- | --- | --- | --- | --- |
| `main` | 右半 | OLED | 右半 | 默认配置 |
| `niceview` | 右半 | nice!view | 右半 | 把屏幕换成 nice!view |
| `leftball` | **左半** | OLED | 左半 | 轨迹球挪到左半，主手也跟着换 |
| `leftball-nv` | **左半** | nice!view | 左半 | 左半球 + nice!view |
| `dualball` | **左右各一个** | OLED | 右半 | 两个轨迹球；用 efogtech 版 pixart 驱动（带 Levels54 参数） |
| `noball` | 无 | OLED | 右半 | 不装轨迹球 |
| `dongle` | 右半 | OLED | **XIAO BLE 接收器** | 用 `keyball61_dongle` + `prospector_adapter` 做中央，左右半变外设 |
| `dongle-nv` | 右半 | nice!view | **XIAO BLE 接收器** | 同上，屏幕换 nice!view |

#### DYA 线分支

| 分支 | 轨迹球 | 屏幕 | 主手（中央） | 说明 |
| --- | --- | --- | --- | --- |
| `dya` | 右半 | OLED | 右半 | 换成 DYA Studio 栈（cormoran 的 ZMK + 驱动 + RPC 模块） |
| `dya-nv` | 右半 | nice!view | 右半 | **推荐**：DYA Studio 栈 + nice!view |

### 推荐分支 `dya-nv` 细节

`dya-nv` 是功能最全的一支：ZMK 用 cormoran 的 `v0.3-branch+dya`
（基于 ZMK v0.3 / Zephyr 3.5），轨迹球驱动换成 DYA 同款的 `cormoran,pmw3610`，
并挂上 DYA Studio 的 custom Studio RPC 模块。其余分支保持原来的配置，没有改动。

已启用的功能：

* **Keymap**：官方 ZMK Studio 键位 / 层编辑
* **Trackball**：运行时可调的输入处理器（速度、旋转、轴吸附、自动鼠标层）
* **Connection**：BLE profile 管理（查看已配对设备、改名、切换、解绑）
* **Settings**：休眠 / 空闲超时等设置
* **电池历史**：记录电量变化曲线

依赖里还带了 default-layer、LED 动画、ext-power-transient、runtime sensor rotate 等
模块（默认未启用），需要时在 `config/boards/shields/keyball_nano/keyball61_right.conf`
里加对应的 `CONFIG_*`。

#### 构建

```sh
make init-standalone   # 依赖下载到 ./dependencies
make build-all         # 输出到 ./build/<目标>/zephyr/zmk.uf2
```

也可以在 GitHub Actions 里跑 `Build ZMK firmware`（`dya-nv` 分支已加入触发列表）。

#### 烧录

| 产物 | 用途 |
| --- | --- |
| `nice_nano_v2__keyball61_right_nice_view_adapter_nice_view.uf2` | 右半 = **主手（中央）**：接 USB / 蓝牙，轨迹球在这一半 |
| `nice_nano_v2__keyball61_left_nice_view_adapter_nice_view.uf2` | 左半 = 副手（外设） |
| `nice_nano_v2__settings_reset.uf2` | 清空已保存的设置（键位改动、配对信息等），从固件默认值重新开始 |

刷法：双击 nice!nano 的复位键进入 UF2 引导模式，把对应的 `.uf2` 拖进去即可。

#### 使用 DYA Studio

1. 用 USB 连接右半（主手）
2. 打开 <https://studio.dya.cormoran.works/>
3. 「Connect via USB」连接键盘

本分支关闭了 Studio 自动锁定（`CONFIG_ZMK_STUDIO_LOCKING=n`），所以不需要解锁键；
想启用锁定就把这一项改成 `y`，并在 keymap 里绑一个 `&studio_unlock`。

#### 轨迹球在键位上的行为

* **自动鼠标层**：轨迹球开始移动 200ms 后自动激活第 4 层（MOUSE），停手 400ms 后退出，
  不用先按层键就能点击 / 滚轮；参数可在 DYA Studio 的 Trackball 页调整。
* **第 5 层（SCROLL）**：轨迹球变成滚轮，速度 1/2。
* **第 6 层（SNIPE）**：轨迹球变成 1/2 速度的精细移动。

#### 注意事项

* 传感器沿用原来的接线与参数（`irq-gpios = <&gpio1 11 …>`、`disable-burst-read`、
  `CONFIG_PM_DEVICE=n`），手感与之前固件一致。
* 电池历史会周期性写入 flash，不需要的话把 `keyball61_right.conf` 里的
  `CONFIG_ZMK_BATTERY_HISTORY*` 关掉。
* `dya-nv` / `dya` 基于 ZMK v0.3 系列，比新一代 `main+dya` 分支（Keyball Neo 47 /
  39 用的那套）少了 runtime macro / combo、OS 检测、device info / watchdog、
  布局预览等功能；需要这些功能的话可以参照 Keyball Neo 仓库的做法迁移。

### 键位图

![keyball61](keymap-drawer/keyball61.svg)

---

## English

[简体中文](#简体中文) &nbsp;|&nbsp; **English**

### About Keyball61

Keyball61 is a 61-key wireless split keyboard: nice!nano controller, built-in
trackball, and a choice of OLED or nice!view display. It belongs to Yowkees' keyball
family, and this repository holds its ZMK config.

Thanks to: PCB [yangxing844](https://github.com/yangxing844), case
[delock](https://github.com/delock), firmware [Amos698](https://github.com/Amos698).

This README is identical on every branch: it introduces the keyboard first, then
explains how the branches differ.

### Branches at a glance

There are two firmware lines. The branch names spell out the differences: `nv` =
nice!view display, `leftball` = trackball moved to the left half, `dualball` = a ball on
both halves, `noball` = no ball, `dongle` = a receiver acts as the central.

* **Base line**: `zmkfirmware/zmk@v0.3` plus the tangbonze build of the PMW3610 driver
  (devicetree compatible `zmk,pmw3610`; CPI, snipe and scroll layers are configured
  through driver options)
* **DYA line**: cormoran's DYA Studio stack — `cormoran/zmk@v0.3-branch+dya`, the
  `cormoran,pmw3610` driver and the custom Studio RPC modules, with live configuration
  from the web UI

#### Base-line branches

| Branch | Trackball | Display | Main hand (central) | Notes |
| --- | --- | --- | --- | --- |
| `main` | right | OLED | right | default configuration |
| `niceview` | right | nice!view | right | same, with a nice!view display |
| `leftball` | **left** | OLED | left | ball moved to the left half, main hand follows |
| `leftball-nv` | **left** | nice!view | left | left ball + nice!view |
| `dualball` | **both halves** | OLED | right | two balls; uses the efogtech build of the pixart driver (Levels54 parameters) |
| `noball` | none | OLED | right | no trackball |
| `dongle` | right | OLED | **XIAO BLE receiver** | `keyball61_dongle` + `prospector_adapter` as the central, both halves as peripherals |
| `dongle-nv` | right | nice!view | **XIAO BLE receiver** | as above, with a nice!view display |

#### DYA-line branches

| Branch | Trackball | Display | Main hand (central) | Notes |
| --- | --- | --- | --- | --- |
| `dya` | right | OLED | right | DYA Studio stack (cormoran ZMK + driver + RPC modules) |
| `dya-nv` | right | nice!view | right | **recommended**: DYA Studio stack + nice!view |

### About the recommended `dya-nv` branch

`dya-nv` is the most capable branch: ZMK comes from cormoran's `v0.3-branch+dya` (based
on the ZMK v0.3 / Zephyr 3.5 line), the trackball uses the same `cormoran,pmw3610`
driver DYA ships, and DYA Studio's custom Studio RPC modules are enabled. The other
branches are unchanged.

What is enabled:

* **Keymap**: official ZMK Studio key/layer editing
* **Trackball**: runtime-configurable input processors (speed, rotation, axis snap,
  auto mouse layer)
* **Connection**: BLE profile management (list, rename, switch, unpair)
* **Settings**: sleep / idle timeouts and other exposed values
* **Battery history**: battery level over time

The manifest also pulls in the default-layer, LED animation, ext-power-transient and
runtime sensor rotate modules (not enabled by default) — add the matching `CONFIG_*` to
`config/boards/shields/keyball_nano/keyball61_right.conf` if you want them.

#### Building

```sh
make init-standalone   # downloads dependencies into ./dependencies
make build-all         # artifacts land in ./build/<target>/zephyr/zmk.uf2
```

Or run the `Build ZMK firmware` GitHub Actions workflow (`dya-nv` is in its trigger
list).

#### Flashing

| Artifact | Purpose |
| --- | --- |
| `nice_nano_v2__keyball61_right_nice_view_adapter_nice_view.uf2` | Right half = **main hand (central)**: USB/BLE to the host, and the trackball |
| `nice_nano_v2__keyball61_left_nice_view_adapter_nice_view.uf2` | Left half = peripheral |
| `nice_nano_v2__settings_reset.uf2` | Wipe stored settings (keymap edits, pairings, …) and start from firmware defaults |

Double-tap the reset button on the nice!nano to enter UF2 bootloader mode, then drag the
matching `.uf2` in.

#### Using DYA Studio

1. Connect the right half (main hand) over USB
2. Open <https://studio.dya.cormoran.works/>
3. Press "Connect via USB"

Studio auto-locking is disabled here (`CONFIG_ZMK_STUDIO_LOCKING=n`), so no unlock key is
needed. To re-enable locking, set it to `y` and bind `&studio_unlock` somewhere.

#### Trackball behaviour in the keymap

* **Auto mouse layer**: after the ball starts moving, layer 4 (MOUSE) is held active and
  released 400 ms after it stops, so clicking/scrolling works without a layer key; both
  delays are adjustable in DYA Studio's Trackball tab.
* **Layer 5 (SCROLL)**: the ball becomes a wheel at 1/2 speed.
* **Layer 6 (SNIPE)**: the ball moves at 1/2 speed for precise pointing.

#### Notes

* The sensor keeps its original wiring and settings
  (`irq-gpios = <&gpio1 11 …>`, `disable-burst-read`, `CONFIG_PM_DEVICE=n`), so the feel
  matches the previous firmware.
* Battery history writes to flash periodically; turn `CONFIG_ZMK_BATTERY_HISTORY*` off in
  `keyball61_right.conf` if you don't need it.
* `dya-nv` / `dya` sit on the ZMK v0.3 line and therefore have fewer features than the
  newer `main+dya` stack used by Keyball Neo 47 / 39: runtime macro / combo, OS
  detection, device info / watchdog and the layout preview are `main+dya` only.

### Keymap

![keyball61](keymap-drawer/keyball61.svg)
