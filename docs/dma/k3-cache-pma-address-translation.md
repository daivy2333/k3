> 来源: https://docs.riscv.org/reference/isa/priv/machine.html（源端修订: Version 1.13；观察日期: 2026-09-09）; https://docs.riscv.org/reference/isa/priv/supervisor.html（源端修订: Version 1.13；观察日期: 2026-09-09）; https://docs.riscv.org/reference/isa/unpriv/rv32.html（源端修订: Version 2.1；观察日期: 2026-09-09）; https://github.com/riscv-non-isa/riscv-iommu（源端修订: Version 1.0；观察日期: 2026-09-09）

# K3 cache、PMA/PBMT 与地址转换

> 本文区分通用 RISC-V 机制、K3 官方概述和固定 revision 第三方实现。DMA 对象及 ownership 状态见 [k3-dma-and-memory-ownership.md](k3-dma-and-memory-ownership.md)；本文不把第三方实现提升为 K3 硬件规范。

## 1. 机制边界

| 机制 | 解决的问题 | 不替代什么 | 本仓库证据 |
| --- | --- | --- | --- |
| cache clean / invalidate | 在 non-coherent 软件路径中发布 CPU 写入，或丢弃 CPU 侧旧副本 | 不定义 ownership、地址翻译或设备完成 | 固定 revision GMAC/UFS 驱动经验 |
| `FENCE` / DMA barrier | 约束指定类别访问的观察顺序 | 不执行 cache clean，不证明 dirty line 已回写 | RISC-V 规范；固定 revision `ov-shm`/UFS |
| PMA | 描述物理区域的访问和内存属性 | 不提供页表地址翻译或设备 domain | RISC-V Privileged 规范；固定 revision OpenSBI 修改 |
| PBMT / Svpbmt | 允许页表 leaf PTE 请求覆盖部分 PMA 属性 | 不等同 PMA，也不保证特定 K3 silicon 接受请求 | RISC-V Supervisor 规范；K3 效果未知 |
| CPU 页表 | 将 CPU VA 翻译到 CPU PA，并附带页级属性 | 不产生设备 IOVA 映射 | RISC-V 通用机制；本文无 K3 页表实例 |
| IOMMU | 按 device/process context 把 IOVA 翻译到系统物理地址并实施隔离 | 不自动提供 cache coherency 或 ownership | RISC-V IOMMU 规范；K3 presence/enablement 未闭合 |

（官方事实）RISC-V Privileged 规范把 PMA 定义为物理地址区域的固有或平台配置属性，并将 PMP 的访问权限检查与 PMA 区分。[Machine-Level ISA](https://docs.riscv.org/reference/isa/priv/machine.html)

（官方事实）Svpbmt 位于 Supervisor-Level ISA；leaf PTE 的 PBMT 字段可以请求 PMA、NC 或 IO 类型。规范允许实现对请求作额外约束，因此“PTE 写入某值”不能单独证明目标平台硬件效果。[Supervisor-Level ISA](https://docs.riscv.org/reference/isa/priv/supervisor.html)

（官方事实）`FENCE` 排序其 predecessor 与 successor 集中的 device I/O 和 memory accesses。它是 ordering 机制，不是 cache-block write-back 指令。[RV32I Memory Ordering Instructions](https://docs.riscv.org/reference/isa/unpriv/rv32.html)

## 2. 地址空间与对象路径

| 对象 | CPU 侧地址 | device/shared-object 中使用的地址 | cache / ordering 路径 | 未闭合边界 |
| --- | --- | --- | --- | --- |
| GMAC descriptor / buffer | 固定第三方驱动持有 CPU 可访问映射 | `bus_addr` 写入 descriptor；ring 的 `dma_addr()` 写入 base/tail 相关寄存器 | 发布 descriptor 前 clean；读取 DMA 回写状态前 invalidate；OWN 发布前有 `dma_wmb` | `bus_addr` 到 CPU PA/IOVA 的映射、默认 Kit IOMMU domain 和硬件 coherency 未知 |
| UFS UTRD/UCD/PRDT / data buffer | DMA allocation 提供 CPU 映射 | `dma_addr()` 的 64 位值拆入 list base、UCD address 和 PRDT DBA/DBAU | data/UCD/UTRD `prepare_for_device`，doorbell 前 `dma_wmb`；完成后 `dma_rmb` 与 `complete_for_cpu` | DMA address 是 PA 还是 IOVA、目标 device domain 和默认 Kit coherency 未知 |
| AP↔RP shared SRAM | 固定工程中 AP 使用主域窗口；RP DTS 可用 local alias | OpenSBI 修改记录的主域窗口为 `0xc0800000..0xc0880000`；RP 注释记录 `0x0..0x80000` local alias | RP `flush()` 仅执行 `fence iorw,iorw`；注释明确它不保证 dirty dcache line 回写 | alias 的硬件翻译、snoop/coherency、双方实际映射属性和真板双向效果未由本项目验证 |

（第三方固定 revision）GMAC `desc.rs` 将 `bus_addr` 拆入 `des0/des1`，`core.rs` 在提交 descriptor 后 clean、回收前 invalidate。该路径说明这个驱动如何维护 descriptor，不证明所有 K3 DMA 都 non-coherent。

（第三方固定 revision）UFS `transfer.rs` 把 `dma_addr()` 写入 UTRL/UTMRL、UCD 和 PRDT，并使用 `prepare_for_device`、`complete_for_cpu`、`dma_wmb`、`dma_rmb`。本文不能由这些 API 名称反推其底层 K3 cache 指令或 IOMMU 映射。

地址关系必须逐段证明，不能由数值相等省略层级：

```text
CPU VA --CPU page table--> CPU PA
                              |
                              +--direct/bypass/identity?--> device address
device IOVA --IOMMU domain?---+

AP main-domain address <--hardware alias?--> RP local address
```

问号表示本项目尚未取得默认 CoM260 Kit 的决定性节点、domain 或运行证据。

## 3. K3 PMA 与 AP/RP 作用域

（第三方固定 revision）OpenSBI `spacemit_k3.c` 的注释和实现为 X100 harts 查找覆盖 `0xc0800000..0xc0880000` 的 PMA entry。在找到 entry 时，代码先执行 `csi_dcache_clean_range()`，把对应 PMACFG byte 改为 `0x22`，再执行 `sfence.vma`；找不到时保留原值并返回。`spacemit_k3_final_init()` 调用此函数，注释说明 PMA CSR 为 per-hart。

这只能证明该固定 OpenSBI 修改对 X100 hart 的软件意图和寄存器写入顺序。作者注释称目标 silicon 忽略 Svpbmt，但本项目没有 K3 官方寄存器资料或独立真板读回，因此不把“Svpbmt 无效”“`0x22` 的硬件效果”写成已验证 K3 事实。

（第三方固定 revision）RP `K3Rt24::init` 明确不访问 custom cache/PMA CSR，因为作者记录 rcpu1 访问会 hang。该事实不表示 AP OpenSBI 会替 RP 配置 PMA，也不证明 RP local alias 的最终属性。

`sfence.vma` 位于 PMA 写之后，只能记录为该补丁选择的同步步骤。它不替代写前的 dcache clean，也不能单独证明外部设备或另一个 hart 已观察到新属性。

## 4. IOMMU 边界

（官方概述）[k3-soc-overview.md](../platform/k3-soc-overview.md) 保存的 K3 官方资料只声明支持 IOMMU 扩展，没有给出默认 CoM260 Kit 的 IOMMU 节点、device attachment、domain 或 map/unmap 路径。

（第三方固定 revision）CoM260 IFX DTS 出现 `iommu-map`、`spacemit,k3-iommu` 和若干 `dma-coherent` 属性。G7 尚未把该变体唯一映射为默认 Kit，因此这些字段只能证明该固定第三方配置存在相应声明。

缺少目标设备的节点、driver、domain 和 map/unmap 路径时，本仓库保持以下字段未知：

- 不推定 IOMMU 不存在或默认启用；
- 不推定 device address 与 CPU PA identity mapping；
- 不推定 bypass、隔离范围或 coherency；
- 不用“DMA 已工作”替代上述证据。

RISC-V IOMMU 规范用于定义 IOVA、device context 和地址翻译术语，不支持任何 K3-specific presence 或默认配置结论。[RISC-V IOMMU Specification](https://github.com/riscv-non-isa/riscv-iommu)

## 5. 未知项

### U1. K3 cache 与 coherency

- 当前证据：官方 DMA 资料未给出 cache line、全局 coherency 或各设备 snoop 范围；固定第三方 GMAC/UFS/共享内存代码只提供局部软件策略。
- 禁止推断：不得把一个驱动的 clean/invalidate 外推到所有设备，也不得假定 K3 全 coherent 或全 non-coherent。
- 解除条件：取得 K3 cache/coherency programmer reference，或取得默认 Kit 上按对象验证的发布、完成和数据可见性结果。
- 影响范围：GMAC/UFS descriptor 与 buffer、共享 SRAM，以及 G5。

### U2. PMA/PBMT 硬件效果

- 当前证据：固定 OpenSBI 修改写 PMA CSR，并记录作者的 Svpbmt 判断；RP 固定代码不访问 custom CSR。
- 禁止推断：不得声称 `0x22` 编码、Svpbmt no-op 或 PMA entry 初值已由官方资料或本项目真板验证。
- 解除条件：取得 K3 PMA/PBMT 寄存器资料和默认固件配置，或取得目标 hart 上的设置、读回及跨端可见性验证。
- 影响范围：AP/RP shared SRAM 的属性与 alias，不外推 GMAC/UFS。

### U3. 默认 Kit IOMMU 与设备地址

- 当前证据：官方概述声明 IOMMU 能力；固定第三方 IFX DTS 含 IOMMU 字段；默认 Kit DTS 仍未唯一映射。
- 禁止推断：不得推定不存在、默认启用、bypass、identity mapping 或特定 device domain。
- 解除条件：唯一确认默认 Kit DTS，并定位 IOMMU node、driver、device attachment、domain 和 map/unmap 路径；必要时补真板地址翻译验证。
- 影响范围：GMAC、UFS、通用 DMA controller 的 device address/IOVA，以及 G5/G7。

## 6. 导航边界

- DMA 对象、ownership 和 device-specific completion：[k3-dma-and-memory-ownership.md](k3-dma-and-memory-ownership.md)。
- K3 官方概述层 IOMMU 能力：[k3-soc-overview.md](../platform/k3-soc-overview.md)。
- G5/G7 与解除条件：[known-gaps.md](../reference/known-gaps.md)。
- AP↔RP 通知不等于数据可见：[com260-mailbox-notification.md](../interrupts/com260-mailbox-notification.md)。
- 后续 network/storage 主题尚未创建，本文不建立前向链接。
