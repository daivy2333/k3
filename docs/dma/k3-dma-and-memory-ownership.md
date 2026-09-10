> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/21-DMA.md（源端修订: unknown；观察日期: 2026-09-02）; https://raw.githubusercontent.com/spacemit-com/docs-buildroot/main/zh/k3_buildroot/device/peripheral_driver/21-DMA.md（源端修订: unknown；观察日期: 2026-09-09）; https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/09-GMAC.md（源端修订: unknown；观察日期: 2026-09-02）; https://raw.githubusercontent.com/spacemit-com/docs-buildroot/main/zh/k3_buildroot/device/peripheral_driver/09-GMAC.md（源端修订: unknown；观察日期: 2026-09-09）; https://www.spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/device/peripheral_driver/ufs.md（源端修订: unknown；观察日期: 2026-09-02）; https://raw.githubusercontent.com/spacemit-com/docs-buildroot/main/zh/k3_buildroot/device/peripheral_driver/ufs.md（源端修订: unknown；观察日期: 2026-09-09）; https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）; https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dts（源端修订: branch k3-br-v1.0.y；观察日期: 2026-09-07）

# K3 CoM260 DMA 与内存所有权事实包

> 范围: 按 D2 区分通用 DMA controller、GMAC / UFS 设备内建 DMA 和 AP↔RP 共享内存三类传输对象；按 D3 把 descriptor 与 data buffer 的 CPU / device ownership 拆分为可观察的五阶段状态机；明确通知（IRQ / doorbell / mailbox）只属于触发或不替代数据可见性。
> 不覆盖: cache maintenance、barrier / fence、PMA / PBMT、IOMMU 与地址转换语义；规划由 `k3-cache-pma-address-translation.md`（Iteration 001，尚未创建）单独交付。
> 边界: 本文档只整合 `21-DMA` / `09-GMAC` / `ufs` 三页官方正文与 [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) / [`k3_com260.dts`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dts) 直接打开的字段；GMAC / UFS 的 descriptor、ring、doorbell、cache 操作细节只引用固定 revision 第三方源码（Rt-Async-AMP `ccb1ff0b` / tgoskits `19219411`），不提升为 K3 官方硬件规范。

## 目录

- [1. 三类传输对象与共同术语](#1-三类传输对象与共同术语)
- [2. 通用 DMA controller](#2-通用-dma-controller)
- [3. GMAC 内建 DMA](#3-gmac-内建-dma)
- [4. UFS 内建 DMA](#4-ufs-内建-dma)
- [5. AP↔RP 共享内存与 mailbox 通知](#5-aprp-共享内存与-mailbox-通知)
- [6. descriptor 与 data buffer 状态机](#6-descriptor-与-data-buffer-状态机)
- [7. 错误、超时、取消与恢复](#7-错误超时取消与恢复)
- [8. 未知项](#8-未知项)
- [9. 与其他主题文档的关系](#9-与其他主题文档的关系)
- [10. 来源与交叉验证导航](#10-来源与交叉验证导航)

---

## 1. 三类传输对象与共同术语

| 传输对象 | 物理路径 | 谁拥有数据 / descriptor | 共同术语（仅作比较，不互替） |
| --- | --- | --- | --- |
| 通用 DMA controller | K3 SoC 集成 DMA 控制器（21-DMA 页面对应） | 设备运行期内由 controller 调度；CPU 准备后归还 controller | channel、descriptor、burst、scatter-gather、cyclic、completion IRQ |
| 设备内建 DMA（GMAC / UFS） | GMAC DWMAC5 描述符 ring；UFS UTRD / UCD / PRDT 队列 | 提交后 CPU 按设备专有完成条件停止访问或回收对应 slot / buffer | ring 索引、GMAC OWN、UFS doorbell/OCS、completion 状态、reclaim |
| AP↔RP 共享内存 | K3 共享 SRAM 区域 + mailbox4 通知 | AP 与 RP 按共享协议约定各通道的生产者和消费者；mailbox 只发门铃 | 共享窗口、alias、PMA 窗口、notification 触发、CPU 重新读取 |

- 边界: 三类对象按本 change 的 D2 分开整理；相同术语不能让官方页面、DTS 与固定 revision 第三方源码互相补值。（计划约束）
- 推论: descriptor 与 data buffer 应分别追踪 CPU/device 访问状态；一方完成转换不能替代另一方的同步条件。（依据 GMAC/UFS 固定 revision 路径与 D3）
- 推论: IRQ、doorbell 或 mailbox 只表明触发或事件；没有对应完成和同步证据时，不能据此认定 CPU 已可见数据。（依据 D3 与固定 revision 路径）
- 事实: CoM260 默认顶层 DTS 尚未唯一映射（G7 `open`），故本文件不假定 `k3_com260.dts` / `k3_com260_kit_v02.dts` 任一为 Kit 默认板值。（未知项；见 [`../reference/known-gaps.md`](../reference/known-gaps.md) G7）

## 2. 通用 DMA controller

- 事实: 21-DMA 官方 URL 已在 R05 和覆盖表登记为 K3 DMA 主题入口；当前只能观察到 SPA 壳，正文状态为 `partially-observed`，因此该 URL 本身不承担具体控制器字段。（官方来源入口）
- 事实: 21-DMA GitHub 对应页（[`21-DMA.md`](https://raw.githubusercontent.com/spacemit-com/docs-buildroot/main/zh/k3_buildroot/device/peripheral_driver/21-DMA.md)）描述 K3 SDK buildroot DMA 控制器与 DMA 通道、CoM260 平台 DMA 节点对应关系；只作 cross-validation，不替代官网。（交叉验证；本 Cycle 2026-09-09 直接打开）
- 事实: K3 顶层 DTSI [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) 未直接登记 `dmac` / `dma-controller` 节点；本仓库未找到 K3 公开的通用 DMA controller binding。（交叉验证；本仓库未直接观察到 K3 通用 DMA binding 是已登记事实在 `k3.dtsi` 之外的另一条事实）
- 事实: linux-6.18 `Documentation/devicetree/bindings/dma/` 目录存在 `adi,axi-dmac.yaml` / `allwinner,sun4i-a10-dma.yaml` / `apple,admac.yaml` 等通用 DMA binding，但无 `spacemit,k3-dma.yaml` 专属 schema。（交叉验证；2026-09-09 通过 GitHub API 检索 `k3-br-v1.0.y` 分支确认）
- 推论: 通用 DMA controller 的完整字段（controller base、channel 数、descriptor 布局、burst 类型、地址宽度、scatter-gather / cyclic 支持、completion 中断）在 K3 公开资料中未由本仓库直接打开证实；下游必须以官方 SoC 寄存器手册或对应 driver 源码补齐。
- 边界: 通用 DMA controller 的 K3 专属寄存器语义、cache 一致性、descriptor 格式、ownership 转换规则、cache line 大小、barrier 列表均由 [`known-gaps.md` G5](../reference/known-gaps.md#g5-dma-coherencyiommu-cache-line-与-barrier-规则) 持有，本文件不重写。

## 3. GMAC 内建 DMA

- 事实: 09-GMAC 官方 URL 已在 R05 和覆盖表登记为 K3 GMAC 主题入口；当前只能观察到 SPA 壳，正文状态为 `partially-observed`，因此该 URL 本身不承担具体驱动字段。（官方来源入口）
- 事实: 09-GMAC GitHub 对应页（[`09-GMAC.md`](https://raw.githubusercontent.com/spacemit-com/docs-buildroot/main/zh/k3_buildroot/device/peripheral_driver/09-GMAC.md)）说明 K3 SDK buildroot GMAC DWMAC5 驱动、CoM260 `&eth1` 对应、`phy-mode = "rgmii"` / `reset-gpios` / `snps,reset-delays-us` / `spacemit,clk-tuning-enable` 等字段；只作 cross-validation。（交叉验证；本 Cycle 2026-09-09 直接打开）
- 事实: [`k3_com260.dts`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dts) 中 GMAC 引用 `&eth1`，`max-speed = <1000>`，`phy-mode = "rgmii"`，`snps,reset-gpios = <&gpio 1 5 GPIO_ACTIVE_LOW>`，`snps,reset-delays-us = <0 20000 100000>`，启用 `spacemit,clk-tuning-enable` / `spacemit,clk-tuning-by-delayline`，`spacemit,tx-phase = <47>`，`spacemit,rx-phase = <53>`；PHY 标识 `ethernet-phy-id001c.c916` + `ethernet-phy-ieee802.3-c22`，`reg = 0x1`，启用 `realtek,aldps-enable` / `realtek,clkout-disable` / `realtek,link-poll`。（交叉验证；MS03 Iter 001 / T11 直接打开，2026-09-07）
- 事实: [`k3_com260_kit_v02.dts`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260_kit_v02.dts) 中 GMAC 改用 `&rgmii1`，关闭 realtek 私有属性，增加 `tx-fifo-depth` / `rx-fifo-depth` / `snps,tso` / `snps,force_sf_dma_mode`；PHY `reg = 0x1`；其余 RGMII / tuning 字段与基础款相同。（交叉验证；MS03 Iter 001 / T11 直接打开，2026-09-07）
- 事实: tgoskits `k3_gmac/desc.rs` 以 `des3.OWN` 表示 DMA ownership：先写地址 / 长度，再执行 Release fence 后置 OWN 位；该字段只在 Linux 原生 DWMAC5 描述符上设置；tgoskits `k3_gmac/core.rs` 在提交前 clean descriptor，回收前 invalidate descriptor，并以 DMA 清 OWN 识别完成。（固定 revision 第三方；Rt-Async-AMP `ccb1ff0b`、tgoskits `19219411`）
- 事实: GMAC ring 在 tgoskits 路径中为 `des3.OWN` 维护；CPU 准备 descriptor → 写地址 / 长度 → Release fence → 置 OWN → 提交 doorbell；device 完成后清 OWN → completion IRQ → CPU invalidate → 回收。该流程不区分 device 内部 TX / RX ring 与共享 memory buffer 之间的 ownership 转换，但明确"先写后置 OWN"是数据可见性前提。（固定 revision 第三方）
- 推论: GMAC 的 K3 专属寄存器布局、descriptor 偏移、`OWN` 字段位置、completion cause 寄存器偏移、IRQ cause / mask 编号、`spacemit,clk-tuning-*` 字段与 RGMII phase 寄存器映射均由 [`known-gaps.md` G6](../reference/known-gaps.md#g6-k3-gmac-寄存器descriptor-与-interrupt-ack-的-programmer-reference) 持有；本文件不重写。
- 边界: GMAC 实物 PHY 型号、Kit 板上连接器位置、MDIO 寄存器、CoM260 Kit 是否为单 PHY 单网口由 [`known-gaps.md` G3](../reference/known-gaps.md#g3-k3-com260-实际-gmac-实例与-phy-详情) `partial` 持有。

## 4. UFS 内建 DMA

- 事实: ufs 官方 URL 已在 R06 和覆盖表登记为 K3 UFS 主题入口；当前只能观察到 SPA 壳，正文状态为 `partially-observed`，R06 聚合状态为 `deferred`，因此该 URL 本身不承担具体队列字段。（官方来源入口）
- 事实: ufs GitHub 对应页（[`ufs.md`](https://raw.githubusercontent.com/spacemit-com/docs-buildroot/main/zh/k3_buildroot/device/peripheral_driver/ufs.md)）说明 K3 SDK buildroot UFS 驱动描述、UTRD / UCD / PRDT 与 doorbell 行为、CoM260 UFS 节点对应；只作 cross-validation，不替代官网。（交叉验证；本 Cycle 2026-09-09 直接打开）
- 事实: tgoskits `k3_ufs/transfer.rs` 把 UTRD / UCD / PRDT 准备、doorbell、轮询完成、timeout / controller error、HCE reset 和单次重试分开；当前 UFS 路径不注册 IRQ，以对应 doorbell bit 清零作为完成条件。（固定 revision 第三方）
- 事实: tgoskits UFS 的访问流程为 CPU 准备 UTRD、UCD 与 PRDT 并调用 `prepare_for_device` → 执行 `dma_wmb` 后写 doorbell → 轮询 doorbell bit 清零 → 执行 `dma_rmb` 和 `complete_for_cpu` → 读取 UTRD OCS 与 response UPIU。其结构没有 GMAC 式 ownership bit。（固定 revision 第三方）
- 推论: UFS 的 K3 专属 doorbell、controller error、HCE reset 和 timeout 规则仍受 programmer reference 缺失限制；本仓库未观察到 K3 UFS 专属 binding。不得为该路径补写不存在的 ownership bit。
- 边界: UFS 协议层（MPHY / UniPro / UTP / SCSI 命令）由 [`known-gaps.md` G6 之外](../reference/known-gaps.md) 的设备层主题承担；MS09（存储）保留 UFS 协议扩展。

## 5. AP↔RP 共享内存与 mailbox 通知

- 事实: K3 CoM260 AP↔RP mailbox 双向通知链由 [`com260-mailbox-notification.md`](../interrupts/com260-mailbox-notification.md)（MS05 Iter 001 交付）单独交付；本文档只承载其与 ownership 相关的边界。（来源: 既有 MS05 文档）
- 事实: AP→RP mailbox4 ch0 经 PLIC source 69 通知 RP；RP→AP mailbox4 ch1 经 APLIC source 217 通知 AP。通知本身不携带数据，仅发门铃。（交叉验证；2026-09-09 引用 MS05 Iter 001 既定事实）
- 事实: 固定 revision DTS 让 AP 与 RP 使用不同地址描述同一共享 SRAM 窗口。AP OpenSBI `7a2df08` 在 X100 hart 的 `final_init` 中查找覆盖窗口的 PMA entry并尝试改为 IO；RP `K3Rt24::init` 明确不访问 custom cache/PMA CSR，因为 rcpu1 上相关访问会挂死。（固定 revision 第三方）
- 事实: tgoskits `ov-shm::flush` 只执行 `fence iorw,iorw`，源码明确说明不足以完成 write-back cache clean；AP / RP 共享内存的可见性不能由 mailbox 通知或 fence 单独证明。（固定 revision 第三方）
- 推论: 共享内存的真实可见性需要与各侧实际映射属性和同步操作一致；现有代码只能证明 AP OpenSBI PMA 修改与 RP fence 的调用位置，不能证明 PBMT、PMA 或双向可见性在本项目真板上成立。
- 边界: AP↔RP 共享内存的 PMA / PBMT 详细机制、cache line 大小、coherency 真板结果由 [`known-gaps.md` G5](../reference/known-gaps.md#g5-dma-coherencyiommu-cache-line-与-barrier-规则) 持有；机制正文规划在 `k3-cache-pma-address-translation.md`（Iteration 001，尚未创建）交付。
- 边界: 共享窗口的固定 revision 大小和 AP/RP 地址配置见 R10 分析；`k3-soc-overview.md` 只确认 K3 SoC 具有 512 KB 共享 SRAM，不承担该第三方窗口的运行地址。

## 6. descriptor 与 data buffer 状态机

每条设备路径用以下五阶段状态机描述 descriptor 与 data buffer 的 CPU / device ownership 转换（D3）。`ring` / `queue` 索引在此状态下视为 device 内部状态，不影响五阶段拆分。

```
[1] CPU 准备         [2] CPU 提交        [3] device owns     [4] 完成观察       [5] CPU 回收
descriptor / buffer  →  (publish)       →  (in flight)      →  (completion)   →  (reclaim)
        ↑                                                        │
        └──────────── error / timeout / reset ────────────────────┘
```

- 阶段 1（CPU 准备）: CPU 填 descriptor 字段（地址 / 长度 / 控制位），按设备规则对 data buffer 写内容并执行 cache clean（如果 buffer 已被 CPU cache 命中）。tgoskits GMAC 先写地址 / 长度，再以 Release fence 排序并设置 ownership bit；tgoskits UFS 准备 UTRD、UCD 与 PRDT；tgoskits 共享内存写 buffer 后执行 `fence iorw,iorw`。（固定 revision 第三方）
- 阶段 2（CPU 提交）: 写设备 doorbell 或对应触发寄存器。GMAC 先置 descriptor OWN，再更新 TX/RX tail；UFS 在同步 UTRD/UCD/data buffer 后写 UTP transfer request doorbell。（固定 revision 第三方）
- 阶段 3（device in flight）: CPU 按设备协议停止改写已经提交的 descriptor、slot 或 buffer。GMAC 以 OWN 标记该状态；UFS 以 outstanding doorbell bit 标记 slot 尚未完成。（固定 revision 第三方）
- 阶段 4（完成观察）: GMAC 通过 IRQ/轮询并在 invalidate 后观察 OWN 清零；UFS 观察 doorbell bit 清零后执行 `dma_rmb` 和 `complete_for_cpu`，再读取 OCS/response。通知本身不证明 data buffer 已可见。（固定 revision 第三方）
- 阶段 5（CPU 回收）: CPU 按设备专有完成条件重新读取 descriptor/response、回收 ring slot 或释放 buffer；复用前必须确认相应同步操作和状态检查已完成。（固定 revision 第三方）

- 推论: IRQ / doorbell / mailbox 通知仅在阶段 4 作为触发；通知到达不证明 data buffer 在阶段 4 → 阶段 5 之间已被 CPU 重新可见；任何在阶段 4 立刻读取 data buffer 的代码必须自行保证 cache invalidate 与 barrier（D3 + D4）。
- 事实: 阶段 3 → 阶段 4 使用设备专有完成条件：GMAC 等待 OWN 清零，UFS 等待 doorbell bit 清零后检查 OCS；两者不能互相补值。（固定 revision 第三方）
- 推论: descriptor 与 data buffer 是两个独立对象；CPU 可能只对 data buffer 做 cache maintenance、对 descriptor 不做，反之亦然；任一对象的所有权转换不互替（D3 + D4）。
- 边界: 上述五阶段是 ownership 抽象；具体 cache maintenance、barrier、PMA / PBMT 路径规划由 `k3-cache-pma-address-translation.md`（Iteration 001，尚未创建）单独交付。

## 7. 错误、超时、取消与恢复

- 事实: DMA error、timeout、设备 reset、对端重启会中断阶段 2 / 3 / 4；tgoskits UFS 路径明确把 HCE reset、timeout、controller error、单次重试独立分支，并把不注册 IRQ 作为已知限制。（固定 revision 第三方）
- 事实: tgoskits GMAC 回收路径先 invalidate descriptor；DMA 已清 OWN 后，CPU 读取 error 字段，再用 `desc::clear` 清整个 descriptor并归还 buffer。代码没有在 error 分支主动清或重新置 OWN。（固定 revision 第三方）
- 推论: error / timeout / reset 后，buffer 或 descriptor 是否可复用取决于设备专有完成与恢复路径；没有完成信号或同步证据时必须视为未定，不能用统一的 OWN 操作代替恢复。
- 事实: tgoskits UFS 在 timeout、OCS error 或 controller fatal 时尝试停止 list、重置 HCE、重新启动 link并重写 transfer-list 基址，成功后重试一次；失败则进入 fatal 状态。GMAC reset 后的 ring 重建语义没有由本文件所引源码完整证明。（固定 revision 第三方 + 未知项）
- 边界: 错误码、timeout 计算、reset 序列、跨域恢复在 K3 公开资料中未由本仓库直接观察到；保留为未知项，登记见第 8 节。

## 8. 未知项

### U1. K3 通用 DMA controller 寄存器与 descriptor 布局

- 当前证据: 21-DMA 官方 / GitHub 页面仅说明 K3 SDK buildroot DMA 驱动通用用法；`k3.dtsi` 与 CoM260 DTS 未直接登记 `dmac` / `dma-controller` 节点；linux-6.18 `k3-br-v1.0.y` 分支无 `spacemit,k3-dma.yaml` 专属 binding。
- 禁止推断: 不得由通用 RISC-V DMA API 或 Synopsys DesignWare DMA 通用行为反推 K3 DMA 寄存器布局、channel 数、descriptor 字段、地址宽度或 completion 寄存器偏移；不得假定 K3 通用 DMA 默认 coherent 或 non-coherent。
- 解除条件: 取得 K3 SoC 公开寄存器手册 DMA 章节；或在 linux-6.18 `k3-br-v1.0.y` 分支定位 K3 DMA controller binding（`Documentation/devicetree/bindings/dma/spacemit,k3-dma.yaml` 或同效 schema）。
- 影响主题: `k3-cache-pma-address-translation.md`（Iteration 001，尚未创建）的 DMA/cache 边界；未来 MS07（通用 DMA 与 GMAC 关系，路径待定）；未来 MS09（通用 DMA 与 UFS / SDHC / QSPI 关系，路径待定）。

### U2. K3 DMA 与 cache 一致性、cache line、barrier 规则

- 当前证据: 由 [`known-gaps.md` G5](../reference/known-gaps.md#g5-dma-coherencyiommu-cache-line-与-barrier-规则) 持有；本文件不重写。
- 禁止推断: 不得由 RISC-V 标准 `fence` / `fence.i` / CBO 指令集行为反推 K3 cache 一致性；不得假定 K3 全 coherent 或全 non-coherent；不得由 Linux DMA API 默认行为反推 K3 cache maintenance 顺序。
- 解除条件: 取得 K3 SoC 公开寄存器手册中 cache / coherency / MMU 章节；或在 linux-6.18 `k3-br-v1.0.y` 分支定位 K3 IOMMU / cache 一致性相关 patch；形成 coherency model、IOMMU presence、barrier list 与 ownership 转换条目。
- 影响主题: 全部依赖 DMA / cache 边界的主题（`docs/dma/`、`docs/network/`、`docs/storage/`、`docs/interrupts/` 中设备通知侧）。

### U3. K3 IOMMU 节点、domain、map / unmap 路径

- 当前证据: 21-DMA / 09-GMAC / ufs 页面与 linux-6.18 `k3-br-v1.0.y` 分支 bindings 目录均未直接登记 K3 专属 IOMMU 节点；linux-6.18 `Documentation/devicetree/bindings/iommu/` 目录含 `qcom,tbu.yaml` / `riscv,iommu.yaml` / `renesas,ipmmu-vmsa.yaml` 等通用 / 厂商 schema，无 `spacemit,k3-iommu.yaml` 专属 schema。CoM260 第三方 DTS（`spacemit-k3-com260-ifx.dts`，固定 revision）出现 `dma-coherent` / `iommu-map` / `spacemit,k3-iommu` 字段，但该 DTS 不在 `k3-br-v1.0.y` 分支官方候选内，且 G7 尚未唯一映射默认 Kit DTS。
- 禁止推断: 不得由 tgoskits 第三方 DTS 出现 `spacemit,k3-iommu` 推定 K3 默认启用 IOMMU、bypass 或 identity mapping；不得由设备驱动可工作反推 IOMMU 不存在；不得由 RISC-V IOMMU 标准行为反推 K3 实现。
- 解除条件: 取得 K3 SoC 公开寄存器手册 IOMMU 章节；或在 linux-6.18 `k3-br-v1.0.y` 分支定位 K3 IOMMU 节点与驱动（`spacemit,k3-iommu.yaml` 或同效 schema）；或 CoM260 官方 DTS 显式登记 IOMMU 节点。
- 影响主题: `k3-cache-pma-address-translation.md`（Iteration 001，尚未创建）的 IOMMU/DMA/地址转换边界；未来 MS07（IOMMU 与 GMAC 联动，路径待定）；未来 MS09（IOMMU 与 UFS / SDHC / QSPI 联动，路径待定）；[`../interrupts/`](../interrupts/)（MSI / IMSIC 联动）。

### U4. AP↔RP 共享内存的 K3 公开寄存器与 PMA 真板结果

- 当前证据: 共享内存地址 alias、PMA 窗口、PBMT 设置由 OpenSBI K3 `7a2df08` 第三方修改、`fence iorw,iorw` 等源码行为可读；K3 SoC 公开寄存器手册未由本仓库直接打开证实 K3 PMA / PBMT 完整语义；CoM260 真板双向可见性结果未由本项目运行。
- 禁止推断: 不得由 OpenSBI 第三方修改存在推定 K3 PMA 实际生效；不得由 `fence iorw,iorw` 单独证明 AP→RP 写入对 RP 端可见；不得由地址 alias 相同推定同一物理存储。
- 解除条件: 取得 K3 SoC 公开寄存器手册中 PMA / PBMT 章节；或在 linux-6.18 `k3-br-v1.0.y` 分支定位 K3 PMA / PBMT 驱动路径；或 CoM260 真板双端读写同步结果。
- 影响主题: `k3-cache-pma-address-translation.md`（Iteration 001，尚未创建）的 PMA/PBMT/地址 alias 边界；[`../interrupts/com260-mailbox-notification.md`](../interrupts/com260-mailbox-notification.md)（mailbox 通知与数据可见分离）。

## 9. 与其他主题文档的关系

- 平台控制资源（pinctrl / clock / reset / APBC / CCU）: 由 [`../platform/k3-platform-control.md`](../platform/k3-platform-control.md)（MS04 Iter 000 交付）单独交付；本文件不复制。
- 启动链与镜像 / DTS 候选: 由 [`../boot/com260-boot-chain.md`](../boot/com260-boot-chain.md) / [`../boot/com260-image-and-dts.md`](../boot/com260-image-and-dts.md)（MS03 Iter 001 交付）单独交付；本文件仅引用 `k3_com260.dts` / `k3_com260_kit_v02.dts` 已观察字段，不复制启动链内容。
- 串口 / UART 与 DMA: 由 [`../serial/com260-uart.md`](../serial/com260-uart.md)（MS04 Iter 001 交付）单独交付；本文件不展开 UART 与 DMA 联动。
- 中断 / 时间 / 通知: 由 [`../interrupts/k3-interrupt-and-time.md`](../interrupts/k3-interrupt-and-time.md) / [`../interrupts/com260-mailbox-notification.md`](../interrupts/com260-mailbox-notification.md)（MS05 交付）单独交付；本文件仅引用 mailbox 通知与 ownership 的边界。
- 板级资源 / 模组引脚 / Kit 底板: 由 [`../platform/com260-board-resources.md`](../platform/com260-board-resources.md)（MS03 Iter 000 交付）单独交付。
- 已知缺口: 由 [`../reference/known-gaps.md`](../reference/known-gaps.md) 持有；本文件 U1-U4 与 G5 / G7 一一对应，不创建新 G 条目。
- 来源覆盖: 由 [`../reference/source-coverage.md`](../reference/source-coverage.md) 持有；本文件首行 8 个 URL 全部一对一登记。
- 后续主题（cache / PMA / PBMT / IOMMU / 地址转换）: 规划由 `k3-cache-pma-address-translation.md`（MS06 Iteration 001，尚未创建）单独交付。

## 10. 来源与交叉验证导航

- 官方权威入口: <https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs>（SPA 壳 `partially-observed`，M01 单一权威源；本文件不直接引用，作为索引边界）。
- 官方页面（21-DMA / 09-GMAC / ufs）: 见首行；正文仍为 SPA 壳 `partially-observed`，实际内容以 GitHub 对应页 cross-validation。
- 官方 GitHub 文档: [`docs-buildroot` 21-DMA.md](https://raw.githubusercontent.com/spacemit-com/docs-buildroot/main/zh/k3_buildroot/device/peripheral_driver/21-DMA.md) / [`09-GMAC.md`](https://raw.githubusercontent.com/spacemit-com/docs-buildroot/main/zh/k3_buildroot/device/peripheral_driver/09-GMAC.md) / [`ufs.md`](https://raw.githubusercontent.com/spacemit-com/docs-buildroot/main/zh/k3_buildroot/device/peripheral_driver/ufs.md)（cross-validation）。
- 官方 DTS / 驱动: [`k3.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3.dtsi) / [`k3_com260.dts`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dts) / [`k3_com260_kit_v02.dts`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260_kit_v02.dts) / [`k3_com260.dtsi`](https://raw.githubusercontent.com/spacemit-com/linux-6.18/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit/k3_com260.dtsi)（cross-validation；linux-6.18 `k3-br-v1.0.y` 分支，2026-09-07 / 2026-09-09 直接打开）。
- 固定 revision 第三方源码: Rt-Async-AMP `ccb1ff0b`、tgoskits `19219411`、OpenSBI `7a2df08`；路径与字段由 `.claude/analysis/rt-async-amp-k3-shared-memory.md` (R10) / `rt-async-amp-k3-drivers.md` (R11) / `rt-async-amp-k3-boot-platform.md` (R09) 持有；本文件只引用其作为 evidence，不复制源码。
- 主题文档不复制覆盖表字段；URL 状态变更由 [`source-coverage.md`](../reference/source-coverage.md) 统一登记。
