# Keyball61 · dya-nv

**简体中文** &nbsp;|&nbsp; [English](#english)

---

## 简体中文

### 关于这块键盘

Keyball61 是一块 61 键的无线分体键盘，自带轨迹球和 nice!view OLED 屏幕，属于
Yowkees 的 keyball 系列；本仓库是它的 ZMK 配置。

感谢：PCB [yangxing844](https://github.com/yangxing844)、外壳
[delock](https://github.com/delock)、固件 [Amos698](https://github.com/Amos698)。

### `dya-nv` 分支做了什么

`dya-nv` 分支把 Keyball61 接到 **DYA Studio**（cormoran 的 ZMK Studio 增强版）上：

* ZMK 换成 cormoran 的 `v0.3-branch+dya`（基于 ZMK v0.3 / Zephyr 3.5 系列）
* 轨迹球驱动换成 DYA 同款的 cormoran PMW3610 驱动（devicetree 兼容名
  `cormoran,pmw3610`），与 keyball dya-nv 上游保持一致
* 加上 DYA Studio 的 custom Studio RPC 模块（运行时输入处理器、BLE 管理、
  设置、电池历史）

`main` 分支保持原样（原来的 badjeff PMW3610 驱动配置），没有改动。

| 项目 | main | dya-nv |
| --- | --- | --- |
| ZMK | `zmkfirmware/zmk@main` | `cormoran/zmk@v0.3-branch+dya` |
| Zephyr | 随 ZMK 决定 | `cormoran/zephyr@v3.5.0+zmk-fixes+nrf-half-duplex-uart` |
| 轨迹球驱动 | badjeff `pixart,pmw3610` | cormoran `cormoran,pmw3610` |
| Studio | 官方 ZMK Studio（键位编辑） | 官方功能 + DYA Studio 的 custom RPC |
| 板级 | `nice_nano_v2` + `keyball61` shield | 同左（结构未变） |

### 已启用的功能

* **Keymap**：官方 ZMK Studio 键位 / 层编辑
* **Trackball**：运行时可调的输入处理器（速度、旋转、轴吸附、自动鼠标层）
* **Connection**：BLE profile 管理（查看已配对设备、改名、切换、解绑）
* **Settings**：休眠 / 空闲超时等设置
* **电池历史**：记录电量变化曲线

依赖里已经带了 default-layer、LED 动画、ext-power-transient、runtime sensor rotate
等模块，但默认没有启用；需要时在 `config/boards/shields/keyball61/keyball61_right.conf`
里加上对应的 `CONFIG_*` 即可。

### 构建

```sh
make init-standalone   # 依赖下载到 ./dependencies
make build-all         # 输出到 ./build/<目标>/zephyr/zmk.uf2
```

也可以直接跑 GitHub Actions 里的 `Build ZMK firmware` 工作流。

### 烧录

| 产物 | 用途 |
| --- | --- |
| `nice_nano_v2__keyball61_right_nice_view.uf2` | 右半 = **主手（中央）**：接 USB / 蓝牙，轨迹球在这一半 |
| `nice_nano_v2__keyball61_left_nice_view_adapter_nice_view.uf2` | 左半 = 副手（外设） |
| `nice_nano_v2__settings_reset.uf2` | 清空已保存的设置（键位改动、配对信息等），从固件默认值重新开始 |

刷法：双击 nice!nano 的复位键进入 UF2 引导模式，把上面对应的 `.uf2` 拖进去即可。

### 使用 DYA Studio

1. 用 USB 连接右半（主手）
2. 打开 <https://studio.dya.cormoran.works/>
3. 「Connect via USB」连接键盘

本分支关闭了 Studio 自动锁定（`CONFIG_ZMK_STUDIO_LOCKING=n`），所以不需要解锁键；
如果想让 Studio 自动锁，把这一项改成 `y`，并在 keymap 里绑一个 `&studio_unlock`。

### 轨迹球在键位上的行为

* **自动鼠标层**：轨迹球开始移动 200ms 后自动激活第 4 层（MOUSE），停手 400ms 后退出，
  不用先按层键就能点击 / 滚轮；参数可在 DYA Studio 的 Trackball 页调整。
* **第 5 层（SCROLL）**：轨迹球变成滚轮，速度 1/2。
* **第 6 层（SNIPE）**：轨迹球变成 1/2 速度的精细移动。

### 注意事项

* 传感器沿用原来的接线与参数（`irq-gpios = <&gpio1 11 …>`、`disable-burst-read`、
  `CONFIG_PM_DEVICE=n`），手感与之前的固件一致。
* 电池历史会周期性写入 flash，不需要的话可以把 `keyball61_right.conf` 里的
  `CONFIG_ZMK_BATTERY_HISTORY*` 关掉。
* 这一支基于 ZMK v0.3 系列（`v0.3-branch+dya`），比新一代的 `main+dya` 分支
  （Keyball Neo 47 / 39 用的那套）功能少一些：runtime macro / combo、OS 检测、
  device info / watchdog、布局预览等目前只在 `main+dya` 上有。

### 键位图

![keyball61](keymap-drawer/keyball61.svg)

---

## English

[简体中文](#简体中文) &nbsp;|&nbsp; **English**

### About this keyboard

Keyball61 is a 61-key wireless split keyboard with an integrated trackball and a nice!view
OLED, part of Yowkees' keyball family. This repository holds its ZMK config.

Thanks to: PCB [yangxing844](https://github.com/yangxing844), case
[delock](https://github.com/delock), firmware [Amos698](https://github.com/Amos698).

### What the `dya-nv` branch does

The `dya-nv` branch makes Keyball61 work with **DYA Studio** (cormoran's enhanced ZMK
Studio):

* ZMK moves to cormoran's `v0.3-branch+dya` (based on the ZMK v0.3 / Zephyr 3.5 line)
* the trackball driver becomes cormoran's PMW3610 driver, the same one DYA uses
  (devicetree compatible `cormoran,pmw3610`)
* DYA Studio's custom Studio RPC modules are added (runtime input processors, BLE
  management, settings, battery history)

`main` is left untouched: it keeps the original badjeff PMW3610 configuration.

| Item | main | dya-nv |
| --- | --- | --- |
| ZMK | `zmkfirmware/zmk@main` | `cormoran/zmk@v0.3-branch+dya` |
| Zephyr | whatever ZMK pins | `cormoran/zephyr@v3.5.0+zmk-fixes+nrf-half-duplex-uart` |
| Trackball driver | badjeff `pixart,pmw3610` | cormoran `cormoran,pmw3610` |
| Studio | official ZMK Studio (keymap editing) | official features + DYA Studio custom RPC |
| Board | `nice_nano_v2` + `keyball61` shield | same (structure unchanged) |

### What is enabled

* **Keymap**: official ZMK Studio key/layer editing
* **Trackball**: runtime-configurable input processors (speed, rotation, axis snap,
  auto mouse layer)
* **Connection**: BLE profile management (list, rename, switch, unpair)
* **Settings**: sleep / idle timeouts and other exposed values
* **Battery history**: battery level over time

The manifest also pulls in the default-layer, LED animation, ext-power-transient and
runtime sensor rotate modules; they are not enabled by default — add the matching
`CONFIG_*` to `config/boards/shields/keyball61/keyball61_right.conf` if you want them.

### Building

```sh
make init-standalone   # downloads dependencies into ./dependencies
make build-all         # artifacts land in ./build/<target>/zephyr/zmk.uf2
```

Or run the `Build ZMK firmware` GitHub Actions workflow.

### Flashing

| Artifact | Purpose |
| --- | --- |
| `nice_nano_v2__keyball61_right_nice_view.uf2` | Right half = **main hand (central)**: USB/BLE to the host, and the trackball |
| `nice_nano_v2__keyball61_left_nice_view_adapter_nice_view.uf2` | Left half = peripheral |
| `nice_nano_v2__settings_reset.uf2` | Wipe stored settings (keymap edits, pairings, …) and start from firmware defaults |

Double-tap the reset button on the nice!nano to enter UF2 bootloader mode, then drag the
matching `.uf2` in.

### Using DYA Studio

1. Connect the right half (main hand) over USB
2. Open <https://studio.dya.cormoran.works/>
3. Press "Connect via USB"

Studio auto-locking is disabled here (`CONFIG_ZMK_STUDIO_LOCKING=n`), so no unlock key is
needed. To re-enable locking, set it to `y` and bind `&studio_unlock` somewhere.

### Trackball behaviour in the keymap

* **Auto mouse layer**: after the ball starts moving, layer 4 (MOUSE) is held active and
  released 400 ms after it stops, so clicking/scrolling works without a layer key; both
  delays are adjustable in DYA Studio's Trackball tab.
* **Layer 5 (SCROLL)**: the ball becomes a wheel at 1/2 speed.
* **Layer 6 (SNIPE)**: the ball moves at 1/2 speed for precise pointing.

### Notes

* The sensor keeps its original wiring and settings
  (`irq-gpios = <&gpio1 11 …>`, `disable-burst-read`, `CONFIG_PM_DEVICE=n`), so the feel
  matches the previous firmware.
* Battery history writes to flash periodically; turn `CONFIG_ZMK_BATTERY_HISTORY*` off in
  `keyball61_right.conf` if you don't need it.
* This branch sits on the ZMK v0.3 line (`v0.3-branch+dya`) and therefore has fewer
  features than the newer `main+dya` stack used by Keyball Neo 47 / 39: runtime macro /
  combo, OS detection, device info / watchdog and the layout preview are `main+dya` only.

### Keymap

![keyball61](keymap-drawer/keyball61.svg)
