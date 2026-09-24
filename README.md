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
* **DYA 线**：cormoran 的 DYA Studio 栈，`cormoran/zmk@main+dya`（ZMK main /
  Zephyr 4.1）+ `cormoran,pmw3610` 驱动 + custom Studio RPC 模块，
  可以在网页上在线调参（含运行时宏与组合键）

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
| `dya` | 右半 | OLED | 右半 | DYA Studio 栈（`main+dya` + cormoran 驱动 + RPC 模块，含宏 / 组合键） |
| `dya-nv` | 右半 | nice!view | 右半 | **推荐**：同上，屏幕换 nice!view |

### 推荐分支 `dya-nv` 细节

`dya` / `dya-nv` 是功能最全的两支：ZMK 换成 cormoran 的 `main+dya`
（ZMK main / Zephyr 4.1），轨迹球驱动是 DYA 同款的 `cormoran,pmw3610`，
并挂上 DYA Studio 的 custom Studio RPC 模块（含运行时宏与组合键）。
基線分支（`main` / `niceview` / `leftball*` / `dualball` / `noball` / `dongle*`）
仍然是原来的 ZMK v0.3 + tangbonze 驱动配置，没有改动。

已启用的功能：

* **Keymap**：官方 ZMK Studio 键位 / 层编辑，布局预览里会画出轨迹球；
  **Macro / Combo** 子页可以创建运行时宏与组合键
* **Trackball**：运行时可调的输入处理器（速度、旋转、轴吸附、自动鼠标层）
* **Connection**：BLE profile 管理（查看已配对设备、改名、切换、解绑）、
  OS 自动识别、按连接 / OS 切换默认层
* **Settings**：休眠 / 空闲超时等设置、通用 custom settings
* **电池历史**：记录电量变化曲线
* **诊断**：device info、watchdog 重启原因

#### 运行时宏与组合键

* **宏**：在 DYA Studio 的 Macro 页新建宏，会分配到槽位号（0 ~ 7）。用
  `&rmacro <槽位号>` 播放：可以写进 keymap，也可以直接在 DYA Studio 的 Keymap 页把
  某个键改掉。当前固件已经 `#include <behaviors/runtime_macro.dtsi>`，两种方式都能用，
  空槽位按下去没有任何动作。
* **组合键**：在 Combo 页按槽位编辑「哪几个键位同时按下 → 触发什么行为」。固件里
  **没有预置任何组合键**，所以添加之前键盘行为不变；全局 timeout / slow-release /
  require-prior-idle 也在该页设置。
* 默认上限：8 个宏（每个最大 256 字节）、8 个组合键（每个最多 16 个键位），要更多就
  改 `keyball61_right.conf` 里的 `ZMK_RUNTIME_MACRO_*` / `ZMK_RUNTIME_COMBO_*`。

#### 构建

```sh
make init-standalone   # 依赖下载到 ./dependencies
make build-all         # 输出到 ./build/<目标>/zephyr/zmk.uf2
```

也可以在 GitHub Actions 里跑 `Build ZMK firmware`（`dya-nv` 分支已加入触发列表）。

#### 烧录

| 产物 | 用途 |
| --- | --- |
| `keyball61_right.uf2`（dya / dya-nv 的产物名） | 右半 = **主手（中央）**：接 USB / 蓝牙，轨迹球在这一半 |
| `keyball61_left.uf2` | 左半 = 副手（外设） |
| `keyball61_reset.uf2` | 清空已保存的设置（键位改动、配对信息等），从固件默认值重新开始 |

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

* 传感器沿用原来的接线与参数（`irq-gpios = <&gpio1 11 …>`、`disable-burst-read`），
  手感与之前固件一致。
* 电池历史会周期性写入 flash，不需要的话把 `keyball61_right.conf` 里的
  `CONFIG_ZMK_BATTERY_HISTORY*` 关掉。
* `dya` / `dya-nv` 已经从 ZMK v0.3 栈迁到 `main+dya`（ZMK main / Zephyr 4.1），
  基線分支与 dya 線的设置存储布局不同：**两边来回刷时，先刷一次
  `keyball61_reset.uf2`**，否则可能出现旧设置干扰。
* 基線分支依旧是 tangbonze 版 PMW3610 驱动（`zmk,pmw3610`），CPI / snipe / 滚轮层
  参数写在驱动配置里；dya 線改由 cormoran 驱动 + DYA Studio 在线调整。

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
* **DYA line**: cormoran's DYA Studio stack — `cormoran/zmk@main+dya` (ZMK main /
  Zephyr 4.1), the `cormoran,pmw3610` driver and the custom Studio RPC modules
  (including runtime macro and combo), with live configuration from the web UI

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
| `dya` | right | OLED | right | DYA Studio stack (`main+dya` + cormoran driver + RPC modules, macro/combo included) |
| `dya-nv` | right | nice!view | right | **recommended**: same, with a nice!view display |

### About the recommended `dya-nv` branch

`dya` and `dya-nv` are the most capable branches: ZMK comes from cormoran's `main+dya`
(ZMK main / Zephyr 4.1), the trackball uses the same `cormoran,pmw3610` driver DYA ships,
and DYA Studio's custom Studio RPC modules (runtime macro and combo included) are
enabled. The base-line branches (`main` / `niceview` / `leftball*` / `dualball` /
`noball` / `dongle*`) keep the original ZMK v0.3 + tangbonze driver configuration.

What is enabled:

* **Keymap**: official ZMK Studio key/layer editing, with the trackball drawn in the
  layout preview; the **Macro / Combo** tabs create runtime macros and combos
* **Trackball**: runtime-configurable input processors (speed, rotation, axis snap,
  auto mouse layer)
* **Connection**: BLE profile management (list, rename, switch, unpair), OS detection and
  per-connection / per-OS default layers
* **Settings**: sleep / idle timeouts and other exposed values, generic custom settings
* **Battery history**: battery level over time
* **Diagnostics**: device info and watchdog reset reasons

#### Runtime macro and combo

* **Macros**: a macro created in DYA Studio's Macro tab gets a slot number (0–7) and is
  played with `&rmacro <slot>` — either written into the keymap or bound from DYA
  Studio's Keymap tab. The firmware already includes
  `behaviors/runtime_macro.dtsi`, so both work; an empty slot does nothing.
* **Combos**: the Combo tab edits, per slot, which key positions trigger which behaviour.
  **No combo ships in firmware**, so behaviour is unchanged until you add one; the
  global timeout / slow-release / require-prior-idle settings live there too.
* Defaults: 8 macros (256 bytes each) and 8 combos (16 positions each) — tune them with
  `ZMK_RUNTIME_MACRO_*` / `ZMK_RUNTIME_COMBO_*` in `keyball61_right.conf`.

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
| `keyball61_right.uf2` (dya / dya-nv artifact name) | Right half = **main hand (central)**: USB/BLE to the host, and the trackball |
| `keyball61_left.uf2` | Left half = peripheral |
| `keyball61_reset.uf2` | Wipe stored settings (keymap edits, pairings, …) and start from firmware defaults |

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
  (`irq-gpios = <&gpio1 11 …>`, `disable-burst-read`), so the feel matches the previous
  firmware.
* Battery history writes to flash periodically; turn `CONFIG_ZMK_BATTERY_HISTORY*` off in
  `keyball61_right.conf` if you don't need it.
* `dya` / `dya-nv` have moved from the ZMK v0.3 line to `main+dya` (ZMK main /
  Zephyr 4.1). The settings storage layout differs from the base-line branches, so
  **flash `keyball61_reset.uf2` first when switching between the two lines**.
* The base-line branches still use the tangbonze PMW3610 driver (`zmk,pmw3610`) with
  CPI / snipe / scroll-layer options in the driver config; the DYA line uses the
  cormoran driver with live configuration from DYA Studio.

### Keymap

![keyball61](keymap-drawer/keyball61.svg)
