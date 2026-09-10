> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/09-GMAC.md（源端修订: unknown；观察日期: 2026-09-02）；https://raw.githubusercontent.com/spacemit-com/docs-buildroot/main/zh/k3_buildroot/device/peripheral_driver/09-GMAC.md（源端修订: unknown；观察日期: 2026-09-09）；https://github.com/spacemit-com/linux-6.18/blob/k3-br-v1.0.y/drivers/net/ethernet/stmicro/stmmac/dwmac-spacemit-ethqos.c（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-09）；https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260_kit_v02.dts（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）

# K3 GMAC DMA ring 与 IRQ

> 范围: CoM260 `eth1` 使用的 DWMAC5 MAC/MTL/DMA 分层，以及固定 revision 第三方实现中的 descriptor、data buffer、doorbell、IRQ 和回收路径。
> 不覆盖: PHY/MDIO/RGMII 静态链、驱动设计、异步 NIC、EtherCAT/TSN 协议和网络栈。
> 证据边界: 官网是唯一权威入口；SpacemiT 官方 GitHub 只作交叉验证；tgoskits `19219411d5dc1515496f910d04c93da12ee95be4` 与 Rt-Async-AMP `ccb1ff0b487e4f49ea570c41f330741eecece935` 只说明固定 revision 第三方行为。

## 目录

- [1. 可证范围](#1-可证范围)
- [2. MAC、MTL 与 DMA 分层](#2-macmtl-与-dma-分层)
- [3. descriptor 与 data buffer](#3-descriptor-与-data-buffer)
- [4. TX ownership 状态](#4-tx-ownership-状态)
- [5. RX ownership 状态](#5-rx-ownership-状态)
- [6. cache、地址与 doorbell 边界](#6-cache地址与-doorbell-边界)
- [7. IRQ、轮询与回收](#7-irq轮询与回收)
- [8. 错误与恢复](#8-错误与恢复)
- [9. 并发推进风险](#9-并发推进风险)
- [10. 未知项](#10-未知项)
- [11. 相邻主题与来源](#11-相邻主题与来源)

## 1. 可证范围

官方 09-GMAC 页面把 K3 网络控制器关联到 DWMAC5 驱动，CoM260 Kit V02 DTS 为 `eth1` 提供 FIFO、TSO 和 store-and-forward 配置。K3 Linux glue driver 直接匹配 `spacemit,k3-gmac`，解析 K3 APMU、interface/delay-line 和 clock tuning 后，把通用数据面交给 stmmac platform probe。（交叉验证）

这些官方材料能证明 K3 glue 与通用 stmmac 数据面的责任边界，但没有给出 K3 GMAC 完整 descriptor、DMA channel status、interrupt cause/mask/ack 或复位值。下文的 ring 与 IRQ 细节来自固定 revision tgoskits，不提升为 K3 programmer reference。

```text
CoM260 eth1 configuration
  → K3 platform glue
  → DWMAC MAC
  → MTL queue
  → DMA channel 0
  → TX/RX descriptor ring + data buffers
  → IRQ or active reclaim
```

PHY、reset 和 RGMII 输入见 [`com260-gmac-phy.md`](com260-gmac-phy.md)。

## 2. MAC、MTL 与 DMA 分层

| 层 | 固定 revision 第三方实现中的职责 | 不能推出的结论 |
| --- | --- | --- |
| K3 glue | APMU interface mode、delay-line、clock/reset 前置；随后建立 MMIO 和 DMA device | 不证明所有 CoM260 DTS 使用同一 phase、clock 或 reset 顺序 |
| MAC | MAC 地址、speed/duplex、TX/RX enable、packet filter 和 checksum 配置 | 不构成 K3 MAC 寄存器规范 |
| MTL | 单 TX/RX queue 的 FIFO、store-and-forward、RX flow control 阈值 | 不证明 Kit V02 DTS 的提示字段等于运行时寄存器已生效 |
| DMA | channel 0、ring base/length、bus mode、TX/RX start、tail pointer、status/interrupt enable | 不证明第三方位定义、复位值或错误恢复适用于全部 SDK revision |
| queue adapter | `ITxQueue`/`IRxQueue` submit/reclaim 与 IRQ `Event` | 不证明异步等待、取消、deadline 或防丢唤醒契约 |

tgoskits 配置为单 queue/channel：ring 各 64 槽，buffer 上限 2048 bytes，ring 对齐 0x1000。以上均为固定 revision 第三方选择，不是官网公开能力上限。

## 3. descriptor 与 data buffer

固定 revision `desc.rs` 把一个 DWMAC4/5 descriptor 表示为 4×32-bit 字段，并额外填充到 64 bytes。其状态和 buffer 必须分开观察：

| 对象 | CPU 准备 | DMA 拥有 | 完成观察 | CPU 回收 |
| --- | --- | --- | --- | --- |
| TX descriptor | 写 buffer 地址、长度、FIRST/LAST/IOC，再置 OWN | DMA 读取并回写状态 | invalidate 后观察 OWN 清除和 error summary | 清 descriptor，推进 `tx_clean` |
| TX data buffer | 上层提供 bus address 与长度 | 从提交到完成期间不得由 CPU 改写 | descriptor 完成后地址进入 `tx_done` | 上层从 reclaim 取得地址 |
| RX descriptor | 写空 buffer 地址，置 BUF1V/IOC/OWN | DMA 写包长度与状态 | invalidate 后检查 OWN、FIRST/LAST 和 error summary | 清 descriptor，推进 `rx_next` |
| RX data buffer | 上层提交可接收区域 | DMA 写入 packet data | descriptor 成功时记录长度；错误时长度为 0 | 上层取得 `(bus_addr, len)` 后决定重投 |

descriptor OWN 只表示 descriptor 的 CPU/DMA 交接。它不能代替 data buffer 的 cache 可见性，也不能给出错误后的设备专有恢复条件。（固定 revision 第三方 + 推论）

## 4. TX ownership 状态

```text
Free
  → CPU writes address/length/FIRST/LAST/IOC
  → Release fence
  → CPU sets OWN
  → descriptor cache clean
  → tail pointer write + ST rewrite
  → DMA in-flight
  → IRQ or active reclaim
  → descriptor invalidate
  → OWN cleared? no: remain in-flight
               yes: check error, clear descriptor, enqueue tx_done
  → CPU reclaims buffer address
```

`submit_tx` 会先主动回收已完成槽。当前 `tx_next` 对应槽仍有 tracked buffer 或 OWN 仍置位时返回 `Retry`，不会覆盖 in-flight descriptor。写 tail 后额外重写 ST，是该实现针对 TBU/fetch 推进的处理，不是已验证的通用 DWMAC 要求。（固定 revision 第三方）

## 5. RX ownership 状态

```text
Free
  → CPU writes receive buffer address + BUF1V/IOC
  → Release fence
  → CPU sets OWN
  → descriptor cache clean
  → RX tail pointer write + SR rewrite
  → DMA in-flight
  → IRQ or active reclaim
  → descriptor invalidate
  → OWN cleared? no: remain in-flight
               yes: validate FIRST/LAST/error and length
  → enqueue (bus_addr, len); error uses len=0
  → CPU reclaims and may resubmit buffer
```

当前槽已有 tracked buffer 或 OWN 仍置位时，`submit_rx` 返回 `Retry`。固定 revision 实现无论 descriptor 成功或出错都会把 buffer 归还上层；`len=0` 只表达该实现的错误结果，不是所有 DWMAC 驱动的统一恢复语义。

## 6. cache、地址与 doorbell 边界

### 6.1 cache 与 fence

固定 revision 实现使用 Release fence 确保地址、长度和控制字段先于 OWN 写入；随后对 descriptor 执行 clean。读取 DMA 回写状态前执行 invalidate。

这条路径只直接展示 descriptor cache 维护。data buffer 的方向性 clean/invalidate、K3 cache line 实值、alias、IOMMU domain 和 CPU address→device bus address 映射仍由 G5 保留。不能因为类型名、64-byte padding 或源码注释就声明 CoM260 全局 coherent/non-coherent 模型已经确认。

### 6.2 地址宽度

tgoskits 将 descriptor 地址拆成低/高 32-bit，并为 DMA device 设置 64-bit mask；注释根据硬件 feature 推导 40-bit addressing 和 EAME 需求。（固定 revision 第三方）

该选择不能证明 K3 全部 GMAC instance 或目标 Kit 的 DMA aperture、IOMMU bypass 和可达物理地址范围。官方 K3 glue driver 本轮只证明 platform handoff，不闭合地址转换。

### 6.3 doorbell

TX/RX 分别写 channel tail pointer，随后重写 ST/SR。doorbell 只能通知 DMA 检查新 descriptor；它不执行 cache clean，不证明 DMA 已读到 descriptor，也不表示 packet 已完成。（固定 revision 第三方 + 推论）

## 7. IRQ、轮询与回收

固定 revision 控制流为：

```text
read DMA channel status
  → neither NIS nor AIS: Event::none
  → otherwise write observed status back (W1C)
  → FBE: log warning
  → reclaim TX, then RX
  → TI/TBU or tx_done non-empty: TX event
  → RI/RBU or rx_done non-empty: RX event
```

IRQ mask 在 enable 时写入 NIS/AIS、FBE、RI、TI 对应位，disable 时清零。TBU/RBU 会参与事件分类，但是否在 mask 中直接启用取决于固定 revision 常量和 DWMAC 汇总位行为；本文件不把该组合提升为官方硬件契约。

`reclaim_tx_buffer` 与 `reclaim_rx_buffer` 也会主动扫描 ring，所以 IRQ 并非唯一完成观察入口。IRQ 提供唤醒/推进信号；完成仍由 invalidate 后读取 descriptor 状态决定。

APLIC source→IMSIC 的上游静态路径见 [`k3-interrupt-and-time.md`](../interrupts/k3-interrupt-and-time.md)。source→EID、target hart、affinity 和真板 delivery 未由本项目验证。

## 8. 错误与恢复

| 条件 | 固定 revision 第三方行为 | 保留边界 |
| --- | --- | --- |
| ring slot busy | submit 返回 `Retry` | 未定义等待、公平性、取消或 deadline |
| TX descriptor error | 记录 warning，清 descriptor 并归还 buffer | 未按 error subtype 给出 packet retry/recovery |
| RX descriptor error | 记录 warning，以 `len=0` 归还 buffer | 上层丢弃、统计和重投契约未在本主题定义 |
| TBU/RBU | 生成对应 queue event；提交时重写 ST/SR | 未证明一次重写能覆盖所有停滞状态 |
| FBE | W1C status、记录 warning，仍执行 TX/RX reclaim | 未观察 channel reset、ring rebuild 或设备隔离 |
| DMA soft reset timeout | 记录 warning 后继续 stop DMA / 重配 MAC/MTL/ring / 启动 DMA，最终返回 `Ok(())` | 继续执行不证明 reset、残留状态或最终数据面可用性恢复；固定 spin 次数不是稳定墙钟 deadline |
| U-Boot 残留 DMA 状态 | 初始化先做 soft reset；源码注释记录 fetch/FTQ 残留风险 | 历史诊断线索，不证明当前板每次存在残留 |
| link-down | 初始化可继续使用静态 speed 线索 | 不证明数据面安全可用；重新协商与恢复属于 PHY/runtime 缺口 |

设备错误不能统一简化为“清 OWN 后重投”。FBE、MTL queue 状态、PHY link 和 descriptor error 属于不同层，恢复动作及停止条件必须由对应 programmer reference 或运行证据确认。

## 9. 并发推进风险

queue adapter 用同一个 `SpinNoIrq<K3GmacCore>` 保护数据面和 IRQ 状态。普通 submit/reclaim 获取阻塞锁；硬中断 handler 用 `try_lock`，失败时直接返回空事件。（固定 revision 第三方）

该策略避免硬中断在持锁路径上忙等，却留下一个未闭合窗口：若最后一次完成中断到达时锁被占用，handler 没有读取/W1C status，也没有生成 queue event；当前材料没有证明释放锁的线程必定重查 status，或设备必定产生下一次中断。因此只能记录“存在潜在推进风险”，不能写成已验证丢中断，也不能写“等待下一 IRQ 必然安全”。

解除此风险需要可审计的 mask→poll→rearm→recheck 契约，或在目标 CoM260 上覆盖锁竞争、ring wrap、最后一次完成和中断风暴的压力证据。本 change 不设计该契约。

## 10. 未知项

### U1. K3 descriptor 与 IRQ programmer reference

- 当前证据: 官方 09-GMAC、DTS 和 K3 glue driver 能确认 DWMAC5/stmmac 接入与平台责任；descriptor、DMA status/mask/W1C 细节来自固定 revision tgoskits。
- 禁止推断: 不把第三方 `regs.rs` 位定义、descriptor padding 或通用 DWMAC 资料写成 K3 寄存器规范。
- 解除条件: 取得 K3 GMAC programmer reference，或直接核对官方 K3 分支中的 stmmac descriptor/DMA 实现并形成适用版本条目。
- 影响主题: G6、descriptor/IRQ 字段、错误分类与恢复。

### U2. GMAC cache 与 device address 模型

- 当前证据: 固定 revision 实现对 descriptor clean/invalidate，并使用高地址字段；MS06 已分离 ownership、PMA/PBMT 和 IOMMU 概念。
- 禁止推断: 不由第三方类型名、注释、DTS `dma-coherent` 或单一变体推定默认 Kit 的 coherency、cache line、IOMMU 与 DMA aperture。
- 解除条件: 官方 cache/IOMMU/DMA 地址资料，或唯一目标 DTS 加目标板可重复的 descriptor/data buffer 可见性验证。
- 影响主题: G5、TX/RX buffer 生命周期、地址映射和恢复。

### U3. IRQ `try_lock` 后的确定性推进

- 当前证据: handler 抢锁失败返回空事件；数据面 reclaim 可主动扫描，但没有证明所有等待路径都会及时重查。
- 禁止推断: 不宣称已发生丢中断，也不假定下一 IRQ、poll worker 或 submit 一定恢复推进。
- 解除条件: 完整 worker recheck 契约，或覆盖最后事件、锁竞争和 ring wrap 的目标板压力证据。
- 影响主题: G4、IRQ 唤醒、TX/RX completion 和后续异步 NIC 设计。

### U4. Fatal error、reset 与 link 恢复

- 当前证据: 固定 revision 实现：FBE 记录 warning 并在初始化执行 soft reset；`reset_dma()` 自清超时返回 `Err`，`init_hardware()` 捕获并记录 warning 后继续 stop DMA / 重配 MAC/MTL/ring / 启动 DMA / 返回 `Ok(())`；源码注释保留 U-Boot 残留、TBU/ST 和 MTL FTQ 历史线索。
- 禁止推断: 不把 warning、W1C、单次 soft reset 或初始化 `Ok(())` 等同于 channel/ring/PHY 已恢复；记录 warning 后继续初始化不证明 reset、残留状态或最终数据面可用性恢复。
- 解除条件: programmer reference 明确错误状态机，并在目标 CoM260 上验证 reset 后 ring 重建、IRQ rearm 和 link 恢复。
- 影响主题: G6、错误传播、设备生命周期和后续驱动计划。

## 11. 相邻主题与来源

- GMAC、MDIO、PHY 与 RGMII: [`com260-gmac-phy.md`](com260-gmac-phy.md)。
- DMA ownership: [`k3-dma-and-memory-ownership.md`](../dma/k3-dma-and-memory-ownership.md)。
- cache、PMA/PBMT、IOMMU 与地址转换: [`k3-cache-pma-address-translation.md`](../dma/k3-cache-pma-address-translation.md)。
- APLIC/IMSIC: [`k3-interrupt-and-time.md`](../interrupts/k3-interrupt-and-time.md)。
- G3–G7: [`known-gaps.md`](../reference/known-gaps.md)。
- 来源元数据: [`source-coverage.md`](../reference/source-coverage.md)。
- 固定 revision 第三方入口: [`k3_gmac`](../../others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/net/k3_gmac/mod.rs)、[`core.rs`](../../others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/net/k3_gmac/core.rs)、[`desc.rs`](../../others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/net/k3_gmac/desc.rs)、[`queue.rs`](../../others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/net/k3_gmac/queue.rs)。

本文件不把第三方 DHCP/ping、源码注释中的真板测量或通用 stmmac 行为改写成本项目验证结果。
