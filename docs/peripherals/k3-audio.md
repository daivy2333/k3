> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/17-Audio.md（源端修订: unknown；观察日期: 2026-09-02）；supporting: https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md（源端修订: 2026-08-25 V1.8；观察日期: 2026-09-07）, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-rdomain.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-dp0.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-dp1.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3-pinctrl.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dtsi, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dts, https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260_kit_v02.dts, https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_ds.md（源端修订: branch k3-br-v1.0.y / unknown；观察日期: 2026-09-07）

# K3 Audio 控制器、端点、时钟、DMA 与板级边界

本文按 SoC Audio 子系统、AP 域 I2S/SSPA、RCPU 域 I2S/SSPA、sound card / DP-eDP endpoint、clock/reset、DMA、audio power domain 和 CoM260 引脚候选分层整理 K3 Audio。平台 provider 与 pinctrl 责任见[平台控制资源](../platform/k3-platform-control.md)，中断拓扑见[中断与时间](../interrupts/k3-interrupt-and-time.md)，DMA 所有权与 cache 边界见[DMA 与内存所有权](../dma/k3-dma-and-memory-ownership.md)与[cache/PMA/地址转换](../dma/k3-cache-pma-address-translation.md)，CoM260 板级资源见[CoM260 板级资源](../platform/com260-board-resources.md)，DTS 候选范围见[镜像与 DTS](../boot/com260-image-and-dts.md)，术语见[术语表](../reference/terminology.md)，缺口见[已知缺口](../reference/known-gaps.md)。

## 1. 范围与证据

- **官方事实**：K3 datasheet §2.6（音频子系统）描述 6 路全双工 I²S、4 路半双工 I²S（其中两路连接 DP/eDP）和 2 路 DP/eDP 音频，并给出 I²S 格式、声道、位深、采样率、sysclk 模式与 TDM 能力。
- **交叉验证**：SpacemiT Linux 6.18 `k3-br-v1.0.y` 的 `k3.dtsi`（AP 域）、`k3-rdomain.dtsi`（RCPU 域）、`k3-dp1.dtsi`（DP1 端点）与 CoM260 DTS 变体提供 I²S controller、`simple-audio-card`、`#sound-dai-cells`、DMA、clock/reset、power domain 与 pinmux 字段。
- **推论**：只在多个已列事实共同支持时使用，不替代缺失的 codec、route、buffer 生命周期、IRQ/xrun 或 stream 日志。
- **未知项**：本项目没有执行 I²S 收发、DP/eDP 音频播放、codec 探测或板级 probe。

SoC 能力、DTS 节点、模组引脚、Kit 连接器与运行结果属于不同层。任一层存在都不自动证明下一层成立。`status = "okay"`、`compatible` 字符串、sound card 与 endpoint 节点存在不证明音频流、采样格式或 codec 协商已经运行。

## 2. SoC Audio 能力

K3 datasheet §2.6 把 Audio 子系统分成三类：

| 类别 | 数量 | 关键参数 | 等级 |
|---|---|---|---|
| 全双工 I²S | 6 路 | 48 kHz / 16 bit / 2 声道；sysclk 64fs / 128fs / 256fs | 官方事实（datasheet §2.6.2） |
| 半双工 I²S | 4 路（其中 2 路连接 DP/eDP） | 48 kHz / 16 bit / 2 声道；兼容 I²S / Left-Justified / Right-Justified；TDM DSP_A / DSP_B 最高 4 声道、16/32 bit | 官方事实（datasheet §2.6.3） |
| DP/eDP 音频 | 2 路 | 最高 192 kHz / 16/20/24 bit / 2 声道；兼容 I²S / Left-Justified / Right-Justified | 官方事实（datasheet §2.6.4） |

datasheet 描述与官方 DTS 静态资源一致：AP 域 `i2s0`–`i2s5` 共 6 路 I²S 节点（§3），RCPU 域 `ri2s0`–`ri2s3` 共 4 路半双工 I²S（§4.1），其中 `ri2s2`/`ri2s3` 与 DPU0/DPU1 DP/eDP audio 端点连接（§4.2），合计 10 路 I²S/SSPA + 2 路 DP/eDP 音频 + 1 个 audio power domain。`spacemit,k1-i2s` 与 `spacemit,k3-ri2s` 字符串只说明软件绑定，不证明 K3 Audio 寄存器全集与 K1 相同。

## 3. AP 域 I²S/SSPA 控制器与 CPU DAI

`k3.dtsi` AP 域包含 6 个 I²S 节点，标签 `i2s0`–`i2s5`，全部默认 `disabled`。

| 节点 | base | reg size | `dmas`（rx, tx） | `dma-names` | 状态 |
|---|---|---|---|---|---|
| `i2s0@d4026000` | `0xd4026000` | `0x30` | `<&pdma 22>, <&pdma 21>` | `rx, tx` | `disabled` |
| `i2s1@d4026800` | `0xd4026800` | `0x30` | `<&pdma 24>, <&pdma 23>` | `rx, tx` | `disabled` |
| `i2s2@d4027000` | `0xd4027000` | `0x30` | `<&pdma 57>, <&pdma 56>` | `rx, tx` | `disabled` |
| `i2s3@d4027800` | `0xd4027800` | `0x30` | `<&pdma 59>, <&pdma 58>` | `rx, tx` | `disabled` |
| `i2s4@d4041000` | `0xd4041000` | `0x30` | `<&pdma 61>, <&pdma 60>` | `rx, tx` | `disabled` |
| `i2s5@d4041800` | `0xd4041800` | `0x30` | `<&pdma 63>, <&pdma 62>` | `rx, tx` | `disabled` |

共同字段（`i2s0`–`i2s5` 通用）：

- `compatible = "spacemit,k1-i2s"`
- `#sound-dai-cells = <0>`（每节点是单 DAI）
- `clocks` 来自 `&syscon_mpmu` 与 `&syscon_apbc`，`clock-names` 依次为 `sysclk_div`/`sysclk`/`bclk`/`sspa_bus`/`sspa`/`c_sysclk`/`c_bclk`（`i2s1` 缺 `sysclk_div`）
- `resets = <&syscon_apbc RESET_APBC_I2Sn>`，单 reset line
- `assigned-clocks` 锁定 `153.6 MHz base`、`sysclk_src`、`sysclk`、`bclk`、实例 `SYSCLK_SEL`
- `assigned-clock-rates` 默认 `307200000 / 307200000 / 12288000 / 1536000 / 614400000`（`i2s1` 末位 `307200000`）

`#sound-dai-cells = <0>` 表示每节点是单个 DAI；CPU DAI 引用通过 phandle 解析，例如 `&i2s0` 直接作为 DAI 节点，未观察到 dai-link 包装节点。datasheet 注释 `must set to same value when enable multi i2s: 8000/16000/48000Hz` 与 `spacemit,fixed-sample-rate = <48000>` 是参考选项；当前 DTS 中以注释形式存在，未启用。

I²S 节点没有单独 `interrupts` 字段；`dmas` 由 [`pdma`](../dma/k3-dma-and-memory-ownership.md) 提供 tx/rx 通道，完成通知走该 provider 的 APLIC source 72，不在 I²S 节点上重复声明。`pdma1` 使用 source 150，但 `i2s0`–`i2s5` 未引用该 provider。I²S 自身 IRQ、xrun、buffer 生命周期、IRQ 与 PDMA completion 顺序未由当前证据展开。

## 4. RCPU 域 I²S/SSPA、sound card 与 DP/eDP 端点

### 4.1 RCPU 域 I²S/SSPA

`k3-rdomain.dtsi` 包含 4 个 I²S 节点 `ri2s0`–`ri2s3`、4 个 ADMA 节点 `adma0`–`adma3` 和 2 个 `simple-audio-card` 节点 `sound_card_dp0` / `sound_card_dp1`，全部默认 `disabled`。

| 节点 | base | reg 字段 | `dmas` | `dma-names` | 状态 |
|---|---|---|---|---|---|
| `ri2s0@c0883100` | `0xc0883100` | `0x300` + `0x4000`（buf） | `<&adma0 1>, <&adma0 0>` | `rx, tx` | `disabled` |
| `ri2s1@c0883500` | `0xc0883500` | `0x300` + `0x4000` | `<&adma1 1>, <&adma1 0>` | `rx, tx` | `disabled` |
| `ri2s2@c0883900` | `0xc0883900` | `0x300` + `0x4000` | `<&adma2 1>, <&adma2 0>` | `rx, tx` | `disabled` |
| `ri2s3@c0883d00` | `0xc0883d00` | `0x300` + `0x4000` | `<&adma3 1>, <&adma3 0>` | `rx, tx` | `disabled` |

共同字段（`ri2s0`–`ri2s3` 通用）：

- `compatible = "spacemit,k3-ri2s"`
- `#sound-dai-cells = <0>`
- `clocks` 来自 `&syscon_rcpu_i2sctrl` 与 `&syscon_rcpu_sysctrl`（`ri2s2`/`ri2s3` 全来自 `rcpu_i2sctrl`），`clock-names = "func", "bus", "sysclk"`
- `resets` 含 `reset` 与 `reset-sys` 两条线
- APLIC IRQ 由 `admaN` 节点声明（`adma0` 263/264、`adma1` 265/`adma2` 264 类比、`adma3` 266/267），I²S 节点自身不重复声明

`ri2s0`–`ri2s3` 与 `adma0`–`adma3` 是一一对应绑定（`ri2s0` 配 `adma0`、`ri2s1` 配 `adma1`、依此类推），均位于 RCPU 域，与 AP 域 `i2s0`–`i2s5`/`pdma` 不重叠。

### 4.2 sound card 与 DP/eDP endpoint

`k3-rdomain.dtsi` 中两个 `simple-audio-card` 节点把 RCPU 域 I²S 与 DPU 的 DP/eDP audio 端点静态连接：

| sound card | 状态 | `mclk-fs` | CPU DAI | format | endpoint 解析 |
|---|---|---|---|---|---|
| `sound_card_dp0` | `disabled` | `<512>` | `&ri2s2` | `left_j` | `dp0_link_cpu` → `&dp0`（`k3-dp0.dtsi`） |
| `sound_card_dp1` | `disabled` | `<512>` | `&ri2s3` | `left_j` | `dp1_link_cpu` → `&dp1`（`k3-dp1.dtsi`，`#sound-dai-cells = <0>`） |

`dp0`（`k3-dp0.dtsi`）和 `dp1`（`k3-dp1.dtsi`）的 `compatible = "spacemit,inno-dp1"` / `inno-dp0`，都通过 `#sound-dai-cells = <0>` 把自身注册成 audio 端点；DPU0/DPU1 选择与显示能力见[K3 SoC 概述](../platform/k3-soc-overview.md) §5（`dpu0` 支持 MIPI-DSI 或 DP/eDP，`dpu1` 仅支持 DP/eDP，V1.8 明确）。

Sound card 的 `status = "disabled"` 不证明音频流、采样参数或与 codec 的协商已经运行。`mclk-fs = <512>` 与 `format = "left_j"` 是静态配置；运行期是否被 `simple-audio-card` 采纳取决于 `status`、codec 探测与 sound card binding。

`simple-audio-card` 的 CPU endpoint 引用 `&ri2sN`，codec endpoint 引用 `&dp0` 或 `&dp1`，并标记 `playback-only`，因此静态拓扑方向为 `I²S → DP/eDP`。本项目未观察到耳机、扬声器或麦克风等外接模拟 codec 节点；这类连接需要额外的 sound card route 与 codec binding。

## 5. 时钟、复位、DMA、power domain 与 CoM260 引脚候选

### 5.1 时钟与复位

| 域 | clock provider | reset provider | 说明 |
|---|---|---|---|
| AP I²S `i2s0`–`i2s5` | `&syscon_mpmu`（`sysclk_div` / `sysclk` / `c_sysclk` / `c_bclk`） + `&syscon_apbc`（`bclk` / `sspa_bus` / `sspa`） | `&syscon_apbc RESET_APBC_I2Sn` | 平台 provider 见[平台控制资源](../platform/k3-platform-control.md) |
| RCPU I²S `ri2s0`–`ri2s1` | `&syscon_rcpu_i2sctrl`（func/bus） + `&syscon_rcpu_sysctrl`（sysclk） | `&syscon_rcpu_i2sctrl` + `&syscon_rcpu_sysctrl` | 复位线 `reset` + `reset-sys` 两条 |
| RCPU I²S `ri2s2`–`ri2s3` | `&syscon_rcpu_i2sctrl`（func/bus/sysclk） | `&syscon_rcpu_i2sctrl`（reset + reset-sys） | sysclk 全部来自 `rcpu_i2sctrl` |

`assigned-clock-rates` 在 AP 域 `i2s0`/`i2s2`–`i2s5` 锁定 `153.6 MHz base / sysclk_src=307.2 MHz / sysclk=307.2 MHz / bclk=12.288 MHz / 实例 SYSCLK_SEL=614.4 MHz`；`i2s1` 末位 `307.2 MHz`。这些是 DTS 静态锁定，不证明运行期 I²S 输出频率与采样率。

### 5.2 DMA consumer

| I²S | DMA provider | 通道（rx, tx） | 等级 |
|---|---|---|---|
| `i2s0` | `&pdma`（AP） | `22`, `21` | 交叉验证 |
| `i2s1` | `&pdma` | `24`, `23` | 交叉验证 |
| `i2s2` | `&pdma` | `57`, `56` | 交叉验证 |
| `i2s3` | `&pdma` | `59`, `58` | 交叉验证 |
| `i2s4` | `&pdma` | `61`, `60` | 交叉验证 |
| `i2s5` | `&pdma` | `63`, `62` | 交叉验证 |
| `ri2s0` | `&adma0`（RCPU） | `1`, `0` | 交叉验证 |
| `ri2s1` | `&adma1` | `1`, `0` | 交叉验证 |
| `ri2s2` | `&adma2` | `1`, `0` | 交叉验证 |
| `ri2s3` | `&adma3` | `1`, `0` | 交叉验证 |

PDMA 与 ADMA 之间的所有权、cache 维护、descriptor 链管理由[DMA 与内存所有权](../dma/k3-dma-and-memory-ownership.md)统一承担；本文件不复制其模型。Audio consumer 只在 `dmas` / `dma-names` 引用方向，不定义 descriptor 格式、buffer 分配或 IRQ/xrun 边界。

### 5.3 power domain

`k3.dtsi` 电源域控制器声明 `audio: audio@2` 节点（`#power-domain-cells = <0>`）。当前 `k.dtsi`（即 `k3.dtsi`）内 AP 域 I²S、RCPU 域 I²S 与 DP/eDP 节点均未引用此 `audio` 域；`dp1`（`k3-dp1.dtsi`）的 `power-domains` 指向 `&power K3_PMU_LCD1_PWR_DOMAIN`，`dpu1_crtc0` 同样指向 `K3_PMU_LCD1_PWR_DOMAIN`。`audio@2` 域存在不代表任何 I²S 或 sound card 已经接入；当前 audio 域消费者未观察到。

### 5.4 CoM260 引脚候选

CoM260 模组 datasheet §I2S 列出 I2S0 与 I2Sx[5:2] 引脚（`I2S0_SCLK` / `I2S0_LRCK` / `I2S0_TXD` / `I2S0_RXD` + `I2Sx[5:2]_{SCLK,LRCK,TXD,RXD}`）。引脚功能复用表（同 datasheet）显示 I2S0 信号出现在 GPIO[5]_111–GPIO[5]_114（与 `SSP1_*` / `SSPA0_*` 复用），I2S3 信号出现在 GPIO[5]_99–GPIO[5]_102（与 `SSP3_*` / `SSPA3_*` 复用）。40 Pin 双排插针（RPi 兼容）的 GPIO/UART/SPI/**I2S**/I2C 引出也包含 I2S 复用候选。

这些只证明模组引脚与 pinmux 候选；不证明目标 DTS 已经选择该 mux、对应 I²S 节点已 `okay`、连接器已在 Kit 底板走线或外部 codec 已经就位。CoM260 DTS 变体（`k3_com260.dtsi` / `k3_com260.dts` / `k3_com260_kit_v02.dts`）未观察到 `&i2sN` 启用，只观察到 `&adma3` / `&ri2s3` / `&sound_card_dp1` / `&dp1` 启用（DP1 静态显示音频链，见 §6.1）。

`k3-pinctrl.dtsi` 包含 I2S / SSPA 的 pinmux 配置（`K3_PADCONF` macro），与 pinctrl provider 责任见[平台控制资源](../platform/k3-platform-control.md)。复用声明不证明 pinctrl 已被该 I²S 节点引用；CoM260 引脚表不证明 Kit 走线与电气匹配。

## 6. 板级映射与运行边界

### 6.1 CoM260 DTS 变体已观察的静态链

`k3_com260.dtsi` / `k3_com260.dts` / `k3_com260_kit_v02.dts` 当前共同观察到：

- `&dp1` → `pinctrl-0 = <&dp1_3_cfg>`、`status = "okay"`
- `&adma3` → `status = "okay"`
- `&ri2s3` → `status = "okay"`
- `&sound_card_dp1` → `status = "okay"`，`simple-audio-card,name = "snd-dp1"`，`simple-audio-card,mclk-fs = <512>`，dai-link@0 `format = "left_j"`，codec 子节点 `playback-only`、`sound-dai = <&dp1>`

只证明该变体形成一条静态显示音频链：`ri2s3 → sound_card_dp1 → dp1 (DPU1 DP/eDP)`，方向 playback-only。`&sound_card_dp0` / `&ri2s2` / `&dp0` / AP 域 `&i2s0`–`&i2s5` 在该变体中未观察到 `okay`。

CoM260 DTS 变体不唯一（[镜像与 DTS](../boot/com260-image-and-dts.md) §已知缺口 G7 持续约束），`k3_com260_kit_v02.dts` 在该组节点上与 `k3_com260.dts` 一致；其他变体（`k3_com260_ifx*`、`k3_com260_tq*`）未在本次范围内被直接打开。

### 6.2 不能推出的结论

- 不得由 I²S 节点 `status = "disabled"` 反推硬件不存在或不能启用。
- 不得由 `compatible = "spacemit,k1-i2s"` / `spacemit,k3-ri2s` 推定 K3 Audio 寄存器全集与 K1 / K3-ri2s 各自前一版本相同。
- 不得由 AP 域 6 路 I²S 与 RCPU 域 4 路 I²S 的总数推定 datasheet 音频子系统的所有能力（datasheet §2.6 还包括 2 路 DP/eDP 音频和 sysclk 模式）。
- 不得由 `simple-audio-card` 节点、`format = "left_j"` 或 `mclk-fs = <512>` 推定实际采样率、位深或运行时格式。
- 不得由 `sound-dai = <&dp1>` 推定 `dp1` 已经输出音频或与 DPU 链路协商成功。
- 不得由 `&adma3` / `&ri2s3` / `&sound_card_dp1` / `&dp1` `okay` 推定音频已经在真板播放或 codec 已经协商。
- 不得由 CoM260 datasheet 引脚表推定目标 DTS 已经选择该 pinmux、Kit 已经走线或外部 codec 已经存在。
- 不得由 `audio@2` 域存在推定任何 I²S / sound card 已经接入该 power domain。
- 不得推定 RCPU 域 `ri2s0`/`ri2s1`/`ri2s2` 或 AP 域 `i2s0`–`i2s5` 在任何当前 CoM260 DTS 变体中已经被启用。
- 不得把 DP1 静态链的状态当作整个 CoM260 音频栈的状态。

## 7. 错误边界与未知项

### U1：codec、route 与采样参数的真实链路

- 当前证据：`sound_card_dp1` 仅 `sound-dai = <&dp1>`，`playback-only`，无外部 codec 节点；AP 域 I²S 与 RCPU 域 `ri2s0/1/2` 未观察到 `okay`；datasheet 给出 48 kHz / 16 bit / 2 声道（I²S）、最高 192 kHz / 16/20/24 bit（DP/eDP）能力。
- 禁止推断：不得由 `format = "left_j"` 推定运行时采样率或位深；不得由 `playback-only` 推定录音路径不可用；不得由 `mclk-fs = <512>` 推定 sysclk 比例被内核采用；不得在 codec 节点、route 表、buffer 生命周期未观察时宣称音频链路工作。
- 解除条件：取得 buildroot 17-Audio.md 正文或 K3 公开 programmer manual Audio 章节；直接打开目标 Linux `simple-audio-card` / `audio-graph-card` binding；真板 `aplay` / `arecord` 或 `tinymix` 拓扑 dump 证明采样、位深和 route 与预期一致。
- 影响主题：本文件 Audio 主题；[CoM260 板级资源](../platform/com260-board-resources.md) 40 Pin I2S 引出与 [DMA 与内存所有权](../dma/k3-dma-and-memory-ownership.md) PDMA/ADMA 完成可见性。

### U2：AP 域 I²S IRQ、xrun 与 buffer 生命周期

- 当前证据：AP 域 `i2s0`–`i2s5` 节点无 `interrupts` 字段；`dmas` 均引用 `&pdma` 通道，该 provider 使用 APLIC source 72。`pdma1` 使用 source 150，但未被本组 I²S 节点引用。
- 禁止推断：不得由 PDMA 完成通知推定 I²S 自身的 tx empty / rx overrun 状态；不得推定 `simple-audio-card` 或 ALSA `snd_pcm` 已经注册 IRQ handler；不得推定 `spacemit,fixed-sample-rate` 注释项被任何 binding 启用；不得由 `i2s1` 缺 `sysclk_div` 推定多路 I²S 时钟配置已经统一。
- 解除条件：取得 K3 公开 I²S / SSPA programmer manual（IRQ、xrun 行为、buffer 门限）；或 buildroot 17-Audio.md 正文提供 IRQ/xrun 处理描述；或真板 `aplay` 期间记录 PDMA 与 I²S 状态寄存器轨迹。
- 影响主题：本文件 AP 域 I²S；[DMA 与内存所有权](../dma/k3-dma-and-memory-ownership.md) PDMA 完成可见性；[中断与时间](../interrupts/k3-interrupt-and-time.md) AP 域 APLIC source 72。

### U3：RCPU 域 I²S、`adma` 与 sound_card 跨域所有权

- 当前证据：RCPU 域 `ri2s0`–`ri2s3` / `adma0`–`adma3` / `sound_card_dp0` / `sound_card_dp1` 默认 `disabled`；`adma0`–`adma3` APLIC IRQ 263–267；`k3_com260.dts` 只启用 `adma3` / `ri2s3` / `sound_card_dp1` / `dp1`。
- 禁止推断：不得由 AP 域 `pdma` 经验外推到 RCPU 域 `adma`；不得由 RCPU 域 I²S 静态 binding 推定 RT24 firmware 已经接管或 mailbox 通知已经建立（参考 [AMP 共享内存生命周期](../amp/k3-amp-shared-memory-lifecycle.md) 与 [RPC ring 通知](../amp/k3-rpc-ring-notification.md)）；不得由 `sound_card_dp1` `okay` 推定 `ri2s2` / `sound_card_dp0` / `dp0` 同步工作。
- 解除条件：取得 RT24 firmware 中 `adma` / `ri2s` / `simple-audio-card` 初始化或 mailbox 协商描述；直接观察 RT24 firmware 启动日志中 audio 相关条目；真板 `aplay` 期间记录 ADMA 与 mailbox 通知时序。
- 影响主题：本文件 RCPU 域 I²S 与 sound card；[AMP 共享内存生命周期](../amp/k3-amp-shared-memory-lifecycle.md) RCPU 域初始化；[RPC ring 通知](../amp/k3-rpc-ring-notification.md) mailbox 链；[中断与时间](../interrupts/k3-interrupt-and-time.md) AP 域 APLIC source 263–267。

### U4：Buildroot 17-Audio 正文与 K3 Audio 完整寄存器手册

- 当前证据：Buildroot 17-Audio 官网入口已登记，源端修订 `unknown`；正文在 2026-09-12 未能直接取得；`k3_ds.md` 给出 SoC 能力子集（§2.6），未覆盖寄存器布局、errata 与 sound card binding 细节。
- 禁止推断：不得依据标题或通用 Linux/Buildroot Audio 文档补写 K3 专属 driver path、`alsaucm` 配置、`tinymix` 拓扑、`simple-audio-card` binding 细节、`spacemit,fixed-sample-rate` 启用条件、`format = "left_j"` 替代项或 IRQ/xrun 处理。
- 解除条件：直接打开对应官网正文或 SpacemiT 官方 GitHub 等价页，并核对源端修订和适用板型；取得 K3 公开 Audio / SSPA / DP-audio programmer manual。
- 影响主题：本文件全部 Audio 子主题的官方事实等级、软件行为和验证入口。

## 8. 主题边界

- GPIO、PWM、IR-RX 由 [k3-gpio-pwm-ir.md](k3-gpio-pwm-ir.md) 承担；WDT、RTC 由后续 `k3-wdt-rtc.md` 承担，本文件不重复定义。
- pinctrl、clock、reset、APBC/CCU provider 见[平台控制资源](../platform/k3-platform-control.md)；AP/RP 中断与 timer 见[中断与时间](../interrupts/k3-interrupt-and-time.md)；DMA 所有权、cache 维护与 descriptor 链见[DMA 与内存所有权](../dma/k3-dma-and-memory-ownership.md)与[cache/PMA/地址转换](../dma/k3-cache-pma-address-translation.md)；AP/RP 生命周期与共享内存/通知链见[AMP 共享内存生命周期](../amp/k3-amp-shared-memory-lifecycle.md)与[RPC ring 通知](../amp/k3-rpc-ring-notification.md)；DTS 候选与映射见[镜像与 DTS](../boot/com260-image-and-dts.md)。
- DPU0/DPU1 显示接口与 V1.8 修订见[K3 SoC 概述](../platform/k3-soc-overview.md) §5。
- 术语、缺口与总索引在 MS11 最终 Iteration 统一收敛，本轮不提前修改。
