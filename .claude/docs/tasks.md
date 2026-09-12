# Tasks

> 维护者: `openspec-milestone-planner` 负责 `MSxx` 路线; `openspec-docs-maintainer` 负责状态同步。
> 当前状态: MS01-MS10 已完成, MS11-MS12 待办, 无进行中任务, 无已承诺待办。

## Milestone Roadmap (MSxx)

### MS01 — 来源覆盖与主题结构基线

- Status: completed
- Outcome: 建立 K3 官网资料的来源覆盖表、主题目录、文档模板和术语种子表, 使每个已发现页面都有明确职责、优先级和预期落点。
- Rationale: 来源清点、主题归类和写作约束共同决定后续所有聚合文档的可追溯性; 单独完成后即可作为各主题 change 的稳定入口。
- Dependencies: None
- Scope: 清点 R01、R04-R08; 区分首批、未来候选、交叉验证和暂不聚合资料; 建立 `docs/index.md`、`docs/reference/` 以及 platform、boot、interrupts、serial、dma、network、storage、buses、peripherals 等主题位置; 定义事实、推论、未知项、版本和术语的表达规则。
- Non-goals: 不完成全部硬件正文; 不选择 K3 之外的平台; 不创建抓取或校验脚本。
- Workload: 中; 需要人工核对至少 52 篇已观察主题页及尚未完整展开的目录, 并解决来源页到主题文档的多对多映射。
- Stable baseline: 所有已发现资料都能从覆盖表定位到一个主题职责; 新文档可直接使用统一模板, 无需重新决定目录和术语风格。
- Verification boundary: R04-R08 的每个 URL 均被覆盖表引用; 目录符合 D02; 模板满足 M02、M03; 每个文档职责唯一; 仓库内没有新增可执行内容。
- Diagnostic boundary: 失败范围限制为来源漏记、主题归类冲突、模板不合规或链接失效, 不进入具体硬件事实争议。
- Split signals: 若新发现的未展开目录显著扩大清点工作, 将补充清点放入独立 change, 但不拆分本 milestone 的结构基线成果。
- Related changes: `establish-k3-doc-foundation` (2026-09-05 收尾, archived)
- Related references: R01, R03, R04, R05, R06, R07, R08

### MS02 — 来源追踪与人工刷新基线

- Status: completed
- Outcome: 建立适用于纯 Markdown 仓库的版本追踪、链接迁移和人工刷新机制, 使来源变化不会形成静默差异。
- Rationale: 大量正文聚合前必须固定观察日期、版本和失效处理规则; 否则后续无法判断差异来自官网更新还是整理错误。
- Dependencies: MS01
- Scope: 记录页面观察日期、文档版本和 SDK 基线; 定义 changed、moved、removed、unreachable 状态; 明确官网正文与官方 GitHub 交叉验证边界; 建立 refresh change 的人工检查清单; 以至少一组 MS01 来源完成实际复核。
- Non-goals: 不实现自动抓取、页面 diff 或链接检查程序; 不把 GitHub 仓库提升为正文权威源; 不聚合新的设备主题。
- Workload: 中; 需要设计并实际演练跨页面、跨版本的人工追踪闭环。
- Stable baseline: 后续文档均可回答依据哪个来源版本整理、何时复核、链接变化后如何处置; 已通过首次真实 refresh 建立持久字段比较、新字段 baseline、SPA 壳归 `unreachable` 与真正的停止条件四节规则, 以及 R01/R07/R08 七个 URL 的逐 URL 结论。
- Verification boundary: 至少一次 refresh change 达到 `accepted`; R01 的最近观察日期与被复核文档一致; 页面迁移和不可访问情形都有可执行处理步骤。
- Diagnostic boundary: 失败范围限制为页面变化、版本遗漏、链接迁移、访问异常或交叉验证边界错误。
- Split signals: 若官网长期要求认证、阻断访问或无法稳定定位页面, 停止刷新并转 Incident, 不扩大本 milestone。
- Related changes: `establish-k3-source-tracking-baseline` (2026-09-05 收尾, archived): 完成 7 URL 首次真实 refresh (R01/R07 两个 URL `unreachable` (SPA 壳), R08 四个 URL `unchanged` 并建立 K3 路径/分支 baseline); T3/T4 通过 R08 supporting fallback 建立 `交叉验证` 等级 SDK baseline (K3 Buildroot SDK v1.0.0-v1.0.7, 核心组件 OpenSBI 1.6 / U-Boot 2022.10 / Linux 6.18 / buildroot 2025.02.6); 持久字段 vs 新 baseline 字段二分写入 `docs/reference/source-refresh.md`; 9/9 tasks 勾选; Plan Review accepted-by-explicit-waiver (用户对 `.omo/` 越界、Act Response 范围、Act 改写 Plan Context 三处偏差显式豁免)。
- Related references: R01, R07, R08

### MS03 — K3 CoM260 板级与启动事实基线

- Status: completed
- Outcome: 形成面向 StarryOS 目标硬件 K3 CoM260 Kit 的 SoC、模组、底板和启动事实包, 支撑其目标板 bring-up 规划。
- Rationale: StarryOS 已确定使用 K3 CoM260; 后续 platform、IRQ、DMA 和 GMAC 工作都依赖统一且可追溯的板级事实。
- Dependencies: MS01, MS02
- Scope: 聚合 K3 product brief 与 datasheet; CoM260 overview、datasheet、hardware resources 和 user guide; 区分 SoC、CoM260 模组与 Kit 底板资源; 整理 CPU/hart、DRAM、UFS、SPI Flash、TF Card、debug UART、GMAC/PHY 和连接器; 整理 boot media、OpenSBI/U-Boot handoff、镜像与 DTS 使用方式; 提取 Rt-Async-AMP 的 AP/RP 镜像、FIT、内嵌 DTB、PT_LOAD、handshake 和资源所有权经验; 登记缺失的寄存器级资料与 U-Boot K3 分支。
- Non-goals: 不聚合 Pico、RV2768 或 Shelf 正文; 不推定未由官网、DTS 或实机证据确认的装载地址、DRAM 布局、GMAC 实例或 PHY 参数; 不把第三方镜像布局提升为 CoM260 官方启动规范; 不修改 StarryOS。
- Workload: 中到大; 需要协调芯片、模组、底板、SDK 和 DTS 多层资料并消除同名资源歧义。
- Stable baseline: CoM260 的板级事实、官方启动资料和第三方 AP/RP 实践均可从主题文档定位, 无需重新遍历 K3 官网和已研究仓库。
- Verification boundary: SoC、模组和底板事实明确分层; boot、memory、console、network 和 storage 均有来源与未知项; 官方事实、官方源码行为和第三方经验分开表达; CoM260 结论不混入其他 K3 板卡事实。
- Diagnostic boundary: 失败范围限制为 SoC/板级混淆、版本差异、启动链缺口、DTS 缺失或模组与底板资源归属不清。
- Split signals: 若单一 CoM260 主题超过 M02 的 500 行建议上限, 按 boot、memory、board-resources 等文档拆分, 但保持同一 milestone。
- Related changes: `establish-k3-com260-board-boot-baseline` (2026-09-08 收尾, archived): 完成 4 篇产品文档 (`docs/platform/k3-soc-overview.md` 179 行 / `docs/platform/com260-board-resources.md` 303 行 / `docs/boot/com260-boot-chain.md` 202 行 / `docs/boot/com260-image-and-dts.md` 210 行, 均 < 450 行); 51 个唯一 URL 来源覆盖 (MS01 baseline 38 + Iteration 000 新增 6 + Iteration 001 新增 7, 含 4 个 raw DTS 文件); 7 个缺口 (G1-G7) 闭包 (G3 partial, G7 新增, G8 删除); 直接打开 4 个 CoM260 DTS 候选文件 (`k3_com260.dts` / `k3_com260_kit_v02.dts` / `k3_com260.dtsi` / `k3.dtsi`), CMA `0x140000000` + DRAM `0x102000000` 两 cell 解码; 13/13 tasks 勾选, Plan Review accepted; 4 份 Rt-Async-AMP 第三方分析已登记 R09-R12。
- Related references: R01, R03, R04, R05, R08, R09, R12

### MS04 — CoM260 平台控制与串口知识基线

- Status: completed
- Outcome: 形成 CoM260 pinctrl、clock、reset、APBC/CCU、UART 和 console 的可追溯知识包。
- Rationale: 平台控制资源决定 UART 和其他设备能否访问; 将官网事实、官方 DTS/驱动行为与第三方 PXA UART 经验分层整理, 可避免资源域和初始化顺序被混用。
- Dependencies: MS03
- Scope: 聚合目标 UART 实例、AP/RCPU 资源域、MMIO width/stride、FIFO、threshold、clock/reset、pinctrl、IRQ、DTS 字段和 bootloader console 线索; 提取 Rt-Async-AMP/tgoskits 中 PXA UART 的 UUE/OUT2、THRE/TEMT、FIFO、IRQ budget、errata 和初始化顺序。
- Non-goals: 不设计或实现异步 UART、copier、ring、waker、flush/tcdrain、VFS/TTY 或 StarryOS 接口; 不把 Linux 或第三方代码行为提升为硬件规范。
- Workload: 中; 需要跨 datasheet、板卡资料、Buildroot 指南、DTS、官方驱动和第三方仓库对齐同一 UART 实例及其资源域。
- Stable baseline: UART 实例、MMIO、时钟、复位、引脚、中断和 console 关系均可从主题文档定位, AP 与 RCPU 事实不会混用。
- Verification boundary: 每项内容标明来源、版本、板型、资源域和证据等级; 官网事实、官方源码行为、第三方经验、推论与未知项分开表达。
- Diagnostic boundary: 资料问题限制在资源域、寄存器、clock/reset、pinmux、IRQ 引用、console 归属或版本差异。
- Split signals: 若 AP UART 与 RCPU UART 的来源、编程模型或术语无法共享同一文档边界, 在本 milestone 内拆分主题文档, 不新增实现 milestone。
- Related changes: `establish-k3-com260-platform-uart-baseline` (2026-09-08 收尾, archived): 完成 2 篇产品文档 (`docs/platform/k3-platform-control.md` 313 行 / `docs/serial/com260-uart.md` 316 行, 均 < 450 行); 59 个唯一 URL 来源覆盖 (MS01-MS02 baseline 38 + MS03 Iter 000 新增 6 + Iter 001 新增 7 + MS04 Iter 000 新增 8); 10 个缺口 (G1-G10, G3 partial, Iter 001 / T3 新增 G8 / G9 / G10, 原有 G1-G7 状态保持); delta spec 同步到 `openspec/specs/k3-com260-platform-uart-baseline/spec.md`（5 added, 0 removed, 0 modified）; Iteration 001 / Cycle 001-rework 修复 AP 域地址范围离散集（`uart0, uart2..uart9` 步进 `0x100`, `uart10` 单独 `0xd401f000`）、串口正文 T4-T6 完成状态补全、首行 10 URL 覆盖、§8 第三方外链转本地相对路径; 6/6 tasks 勾选, Plan Review accepted, Act Self-Review 0 Critical / 0 Important / 1 Minor（§12 行数硬编码）。
- Related references: R03, R04, R05, R08, R10, R12, R13

### MS05 — CoM260 中断、时间与通知机制知识基线

- Status: completed
- Outcome: 形成 CoM260 AIA、APLIC、IMSIC、timer、mailbox 和设备通知机制的统一知识包。
- Rationale: 中断控制器、时间源和设备通知共享 hart/context、claim/complete 与清除顺序等术语, 但 AP/RP 域和 wired IRQ/MSI 路径不能相互替代。
- Dependencies: MS03
- Scope: 整理 AIA、APLIC、IMSIC 的角色与拓扑, hart/context、IRQ domain、routing、mask、claim/complete、ack/EOI、wired IRQ、MSI/MSI-X、timer、deadline、timeout 和 suspend/resume; 提取 tgoskits 的 `stopei`、重复 IRQ、mailbox FIFO/pending 清除与 self-test 经验。
- Non-goals: 不实现或设计操作系统中断子系统; 不声称 Linux 或第三方实现等同于裸机硬件行为; 不展开设备数据面。
- Workload: 中到大; 需要协调 datasheet、CoM260 DTS、SDK 文档、官方内核来源和 AP/RP 两侧第三方实现。
- Stable baseline: AP/RP 控制器、hart/context、wired IRQ、MSI、claim/complete、ack/EOI、timer 和 mailbox 关系均有可检索说明。
- Verification boundary: 硬件规范、DTS 路由、官方软件行为和第三方经验明确分层; 缺失地址、delivery 与清除规则保留为未知项。
- Diagnostic boundary: 资料问题限制在 IRQ domain、控制器拓扑、hart routing、mask、claim/complete、MSI delivery、timer 或通知清除顺序。
- Split signals: 若 wired IRQ、MSI 与 mailbox 的资料规模或术语冲突使单篇文档难以维护, 在本 milestone 内按机制拆文档。
- Related changes: `establish-k3-com260-interrupt-time-notification-baseline` (2026-09-09 收尾, archived)
- Related references: R03, R05, R07, R08, R10, R11, R12, R14

### MS06 — CoM260 DMA、cache、PMA 与内存所有权知识基线

- Status: completed
- Outcome: 形成通用 DMA、设备内建 DMA、cache coherency、barrier、PMA、PBMT、IOMMU 和地址转换的分层知识包。
- Rationale: 官方资料与第三方实现分别提供能力描述和实际处理顺序; 只有保留证据等级及设备边界, 才能避免把某个控制器的做法泛化为 K3 全局事实。
- Dependencies: MS03
- Scope: 整理 DMA controller、channel、descriptor、burst、地址宽度、scatter-gather、cyclic DMA、cache line、barrier、map/unmap、PMA、PBMT、IOMMU 和地址转换; 建立 CPU/device ownership 与完成可见性问题表; 提取 OpenSBI PMA、Zicbom、GMAC/UFS cache maintenance、共享 SRAM alias 和逐 hart 设置经验。
- Non-goals: 不把 Linux DMA API 当作硬件规范; 不设计 Rust DMA API; 不实现设备驱动; 不把共享内存一致性直接等同于 GMAC、UFS 或其他设备 DMA 一致性。
- Workload: 大; 需要从能力说明、DTS、官方驱动、OpenSBI 补丁和第三方设备实现中分离事实、软件约定、经验与未知项。
- Stable baseline: CPU/device ownership、descriptor/data buffer、物理与设备地址、coherent/non-coherent、PMA/PBMT 和 IOMMU 边界均可查询。
- Verification boundary: 每项结论标明来源、适用设备和证据等级; descriptor/data buffer、CPU/device 与 coherent/non-coherent 边界完整。
- Diagnostic boundary: 资料问题限制在寻址、descriptor、cache、barrier、PMA、PBMT、IOMMU 或完成可见性。
- Split signals: 若通用 DMA、共享内存和设备内建 DMA 无法共享术语或文档规模超过 M02 建议上限, 在本 milestone 内拆分主题文档。
- Related changes: `establish-k3-com260-dma-memory-ownership-baseline` (2026-09-09 收尾, archived): 完成 2 篇产品文档 (`docs/dma/k3-dma-and-memory-ownership.md` 154 行 / `docs/dma/k3-cache-pma-address-translation.md` 79 行, 均 < 450 行); 68 个唯一 URL 来源覆盖 (MS01-MS02 baseline 38 + MS03 Iter 000 新增 6 + Iter 001 新增 7 + MS04 Iter 000 新增 8 + MS05 Iter 000 新增 3 binding + MS06 Iter 000 新增 3 docs-buildroot GitHub + Iter 001 新增 3 RISC-V spec 仓库, 含 3 个 RISC-V spec: riscv-iommu / privileged.adoc / unprivileged.adoc); 10 个缺口 (G1-G10, G3 / G4 / G5 partial, 其余 G1 / G2 / G6 / G7 / G8 / G9 / G10 状态保持); delta spec 同步到 `openspec/specs/k3-com260-dma-memory-ownership-baseline/spec.md`; Iter 000 完成 21-DMA / 09-GMAC / ufs docs-buildroot GitHub 对应页直接打开 + k3-dma-and-memory-ownership.md 创建, Iter 001 完成 k3-cache-pma-address-translation.md 创建（5 段 cache / PMA / PBMT / IOMMU + 1 段地址空间 + 边界段全部按证据强度标注）+ G5 由 open 调整为 partial（PMA 16 entries + Svpbmt K3 silicon 忽略 + 无 RISC-V IOMMU + `fence iorw,iorw` barrier 行为回写证据）+ source-coverage 65 → 68 + index.md URL/缺口计数同步; 5/5 tasks 勾选, Plan Review accepted-by-explicit-waiver（用户对 Gate 2 User Plan Approval 字段显式豁免; Act 自承担 Plan Review 终态; 风险已记录）。
- Related references: R03, R04, R05, R08, R09, R11, R12, R15

### MS07 — CoM260 GMAC、MDIO、PHY 与网络硬件知识基线

- Status: completed
- Outcome: 形成 CoM260 GMAC、MDIO、PHY、RGMII、descriptor、DMA ring 和设备中断的可追溯知识包。
- Rationale: 网络硬件知识同时跨越平台资源、PHY 链路、DMA 和中断; 单独聚合可保留各层来源和故障边界, 又不引入任何操作系统实现计划。
- Dependencies: MS04, MS05, MS06
- Scope: 整理 CoM260 GMAC 实例、MMIO、clock/reset、pinctrl、MDIO、PHY 型号与地址、link、RGMII phase/delay、reset、ref clock、RX/TX descriptor ring、ownership、reclaim、interrupt cause、mask、ack 和 budget; 提取 tgoskits DWMAC5 初始化、PHY/MDIO、DMA ring、IRQ 和抢锁失败风险。
- Non-goals: 不规划或实现 polling、IRQ、async NIC、网络栈或 StarryOS 接口; 不用 QEMU 结果证明 CoM260 硬件行为; 不在缺少证据时预选寄存器模型。
- Workload: 大; 需要把官网资料、CoM260 板级信息、官方 DTS/驱动和第三方 DWMAC5 实现按层整理。
- Stable baseline: GMAC 实例、PHY 链路、MDIO、RGMII、descriptor、DMA ring、IRQ cause 和回收顺序均有来源与未知项说明。
- Verification boundary: 官方事实与第三方实现相互对照但不相互替代; 平台、PHY、MAC、DMA 和 IRQ 内容分层明确。
- Diagnostic boundary: 资料问题限制在板级链路、PHY、MDIO、MAC、DMA ring、IRQ cause、reclaim 或版本差异。
- Split signals: 若 GMAC 寄存器、PHY 和 descriptor 内容超过 M02 建议上限, 在本 milestone 内拆为 hardware、phy-link 和 dma-irq 文档。
- Related changes: `establish-k3-com260-gmac-network-baseline` (2026-09-10 收尾, archived): 完成 2 篇产品文档 (`docs/network/com260-gmac-phy.md` 238 行 / `docs/network/k3-gmac-dma-irq.md` 206 行, 均 < 450 行); 70 个唯一 URL 来源覆盖 (在 MS06 68 之上新增 K3 GMAC glue driver `dwmac-spacemit-ethqos.c` 与既有两篇正文引用的官方 URL, URL 总数 69 → 70); 10 个缺口 (G3 / G4 / G5 = `partial`, G6 / G7 = `open`, G1-G2 / G8-G10 状态保持); delta spec 同步到 `openspec/specs/k3-com260-gmac-network-baseline/spec.md`（5 added, 0 removed, 0 modified）; Iter 000 完成 `com260-gmac-phy.md` 创建 (SoC/模组/Kit/DTS 变体 + `eth1` 平台资源 + MDIO/PHY/RGMII 静态链 + 来源差异/运行时边界 + 5 个四字段未知项), Iter 000 / Cycle 001-rework 修复 PLAN-INVALID（来源任务越界） + ACT-DEVIATION（§4.1 MMIO/IRQ 证据等级越级）2 项 Important 收敛 accepted; Iter 001 完成 `k3-gmac-dma-irq.md` 创建 (MAC/MTL/DMA 分层 + TX/RX 状态机 + descriptor/buffer 分离 + cache/doorbell + IRQ/W1C/reclaim + 错误恢复 + `try_lock` 推进风险 + 4 个四字段未知项) + G3–G7 收敛 + index 接入 (T5 修正 G4 汇总 `open → partial` 与 k3.dtsi 静态拓扑一致) + §8 DMA soft reset timeout (Acceptance 3: `core 初始化返回错误` → `记录 warning 后继续 stop DMA / 重配 / 启动 DMA / 返回 Ok(())`) + U4 同步 reset → warning → 继续初始化语义 + index 维护规则计数 69 → 70 (Acceptance 5) 2 项 Plan Review Important 修复 (Plan Review 复审 accepted); 6/6 tasks 勾选, Persisted Evidence `none`, 未触动 `others/` 或既有 staged 内容。
- Related references: R03, R04, R05, R07, R08, R10, R12, R16

### MS08 — K3 AMP、共享内存、RPC 与跨核通信知识基线

- Status: completed
- Outcome: 形成 K3 AP/RP 生命周期、共享窗口、mailbox、ring、RPC、doorbell 和恢复行为的独立知识包。
- Rationale: Rt-Async-AMP 的核心经验跨越启动、中断与内存域; 独立主题可串联完整调用链, 避免细节散落后失去资源所有权和内存序上下文。
- Dependencies: MS03, MS05, MS06
- Scope: 整理 AP/RP 镜像与握手、共享内存地址和 alias、初始化所有权、PMA/PBMT、mailbox、doorbell、ring、RPC、BUSY、错误响应、multi-block 发布、watchdog re-init 和恢复; 登记 `rt-async`、`ov-channels`、U-Boot K3 分支、手册、原理图与运行日志缺口。
- Non-goals: 不实现 AMP、RPC、ring 或共享内存组件; 不把源码注释和作者历史数据写成本项目验证结论。
- Workload: 大; 已有主仓和 tgoskits 资料可读, 但 RP 启动、ring/原子和 U-Boot 调用链仍受缺失仓库与材料限制。
- Stable baseline: AP/RP 资源域、共享窗口生命周期、数据与通知路径、内存序、错误传播和恢复风险可从统一入口追踪。
- Verification boundary: 已读源码、作者声明、缺失实现、推论和未验证行为分别标注; 每条跨仓库调用链记录提交与分支。
- Diagnostic boundary: 资料问题限制在初始化所有权、地址 alias、PMA/PBMT、ring ordering、通知丢失、错误响应、watchdog re-init 或缺失依赖。
- Split signals: `rt-async` 或 `ov-channels` 补齐后若形成可独立说明的 RP runtime 或 ring/原子知识域, 在本 milestone 内增加子文档。
- Related changes: `establish-k3-amp-rpc-baseline` (2026-09-10 收尾, archived): 完成 2 篇产品文档 (`docs/amp/k3-amp-shared-memory-lifecycle.md` 205 行 / `docs/amp/k3-rpc-ring-notification.md` 246 行, 均 < 450 行); 70 个唯一 URL 来源覆盖 (在 MS07 70 之上不增减, 复用 boot/DTS/标准 URL 并把 `amp` 主题职责加入既有行); 11 个缺口 (G1–G10 + G11, G4/G5/G7 partial 保持 + G11 新增); delta spec 同步到 `openspec/specs/k3-amp-rpc-baseline/spec.md`（6 added, 0 removed, 0 modified）; R17 登记; Iteration 000 / Cycle 000 完成 T1 (3 条既有官方 URL 增加 `amp` 主题职责) + T2 (创建 `k3-amp-shared-memory-lifecycle.md`, 5 个四字段未知项, 11 个相对链接全部有效, 字段说明在 Plan Review 接受后由同一 Cycle 内部补 `amp`); Iteration 001 / Cycle 000 完成 T3-T7 (创建 `k3-rpc-ring-notification.md` 246 行 + G11 + 五项 AMP/RPC/ring/doorbell/共享窗口术语 + index AMP 入口与计数) 后被 Plan Review 标记 `rework-required` (Acceptance 2 BUSY=1 行为错误 + Act Response 重复); Iteration 001 / Cycle 001-rework 修复 T4-R1 (BUSY=1 描述为"服务端处于可发现请求的弹性窗口") + T4-R2 (父 Cycle Act Response 单一化); Iteration 001 / Cycle 002-rework 修复 T4-R3 (三处 BUSY/NOTIFY 表述越界, 全文 BUSY 仅描述为弹性轮询状态提示, NOTIFY 明确为 BUSY=0 条件动作, 不再声称并发保护); 7/7 tasks 勾选, 两轮 rework 均被 Plan Review accepted (002 终态), Persisted Evidence `none`, 未触动 `others/` 或既有 staged 内容, `git diff --check` 退出 0, `openspec validate --strict` 退出 0。
- Related references: R09, R10, R11, R12, R17

### MS09 — K3 存储控制器知识基线

- Status: completed
- Outcome: 形成 QSPI、SPI、SDHC 和 UFS 的控制器、启动关系、数据路径、DMA 与恢复知识包。
- Rationale: 存储主题共享启动介质和 DMA 术语, 但各控制器的 PHY、命令和错误恢复不同; 聚合时需保留设备边界。
- Dependencies: MS03, MS06
- Scope: 聚合官网 QSPI、SPI、SDHC 和 UFS 资料; 整理控制器实例、DTS、clock/reset、pinctrl、IRQ、DMA、启动用途和错误处理; 提取 tgoskits UFS 的 MPHY、UniPro、UTP/SCSI、cache maintenance、fatal error、timeout recovery 和轮询行为。
- Non-goals: 不实现存储驱动; 不把 UFS 经验外推到 SDHC、QSPI 或 SPI; 不把存在 IRQ 描述等同于运行路径使用 IRQ。
- Workload: 中到大; 官网已有四类主题入口, UFS 另有第三方实现经验可交叉整理。
- Stable baseline: 每类存储控制器的职责、资源、启动关系、数据路径和资料缺口均有主题入口。
- Verification boundary: 来源覆盖表中的对应页面都有正文落点; 官方事实、官方软件行为和第三方 UFS 经验分开表达。
- Diagnostic boundary: 资料问题限制在控制器实例、PHY、命令、DMA、IRQ、timeout、recovery 或启动介质关系。
- Split signals: 若 UFS 内容超过 M02 建议上限或其术语无法与其他存储主题共用, 在本 milestone 内单独成文。
- Related changes: `establish-k3-storage-controller-baseline` (2026-09-11 收尾, archived): 完成 2 篇产品文档 (`docs/storage/k3-qspi-spi-sdhci.md` 151 行 / `docs/storage/k3-ufs.md` 153 行, 均 < 450 行); 70 个唯一 URL 来源覆盖 (在 MS08 70 之上不增减, 4 个存储官网入口由 deferred 改为 active, UFS 复用 1 个 docs-buildroot supporting row); 12 个缺口 (G1-G11 + G12, G3/G4/G5 partial 与 G7 open 保持, G12 新增承载存储控制器板级映射 / 运行路径 / 恢复闭包); 15 项术语新增 (SPI / QSPI / SDHCI / eMMC / UFS / M-PHY / UniPro / UTP / UPIU / UTRD / UTMRD / UCD / PRDT / SCSI / LUN); delta spec 同步到 `openspec/specs/k3-storage-controller-baseline/spec.md`（7 added, 0 removed, 0 modified）; R23 登记; Iteration 000 / Cycle 000 完成 T1 (4 个存储官网入口由 deferred 改为 active, 复用 1 个 UFS docs-buildroot supporting row, URL 总数保持 70) + T2 (创建 `k3-qspi-spi-sdhci.md` 151 行, 5 个 U 字段, 8 个首行 URL 覆盖); Iteration 001 / Cycle 000 完成 T3 (创建 `k3-ufs.md` 153 行, 12 段分层: 范围与证据等级 / SoC 能力与板级可达性 / 静态资源 / 启动关系 / MPHY/UniPro/Link startup / UTP/SCSI descriptor / DMA/cache/doorbell / SCSI/LUN scan / 完成模型 / 错误超时与恢复 / 未知项 U1-U5 / 主题边界与导航, 10 个首行 URL 覆盖 + 固定 revision 第三方 `19219411d` 链接 + U1-U5 4 字段); Iteration 002 / Cycle 000 完成 T4-T6 (G12 板级映射 / 运行路径 / 恢复闭包 + 15 项存储术语 + index.md storage 双入口 / 70/40/12 计数 / docs/storage/ 状态由 `待聚合；R06 状态 deferred` 改为 `已聚合(Iteration 000/001)；G5 partial, G7/G12 open`) 后被 Plan Review 标记 rework-required (R1: 10 个 UFS 术语章节定位与数据结构描述错误 / R2: G12 区块 `k3_ufs/transfer.rs` permalink 指向非固定 revision / R3: change `tasks.md` T4-T6 未勾选 / R4: Act 覆盖响应时删除 Plan Review 区域); 同一 Cycle 内 Act Response `reported → pending` 后做有限修复: R1 修正 M-PHY/UniPro → §5 / UTP/UPIU/UTRD/UTMRD/UCD/PRDT → §6 / SCSI/LUN → §8 章节定位与语义 (UTRD 改 command type / data direction / interrupt bit / OCS / UCD base / response UPIU offset+length / PRDT offset+length; UTMRD 改 "只分配并编程, 未观察到 task-management 提交"; UCD 改 "command UPIU + response UPIU 各 512-byte 对齐区域"; PRDT 移除 DBC 20 bit 第三方数值; UPIU 移除 OCS/response 错误归属; SCSI 列表移除 NOP/QUERY 错误项); R2 把 G12 区块 permalink 从 `github.com/rt-async-amp/tgoskits` 替换为 `github.com/PlaticaIt/StarryOS/blob/19219411d.../k3_ufs/transfer.rs` 与 `k3-ufs.md:68,100` 同款; R3 把 change `tasks.md` 第 6-8 行 `[ ]` 改为 `[x]`; R4 覆盖 Act Response 时未触动 `## Plan Review` 区域; 6/6 tasks 勾选, 末轮 Plan Review accepted, `Next Cycle: None` / `Next Iteration: None`, Persisted Evidence `none`, 未触动 `others/` 或既有 staged 内容, `git diff --check` 退出 0, `openspec validate --strict` 退出 0。
- Related references: R04, R05, R08, R10, R12, R20, R23

### MS10 — K3 外设总线知识基线

- Status: completed
- Outcome: 形成 I2C、USB、PCIe、CAN 和 EtherCAT 的控制器能力、资源与板级连接知识包。
- Rationale: 这些主题在当前来源覆盖表中已有官方入口, 但尚无长期正文; 按总线聚合可填补知识库中的设备互连空白。
- Dependencies: MS03
- Scope: 聚合各总线的控制器实例、DTS、clock/reset、pinctrl、IRQ、DMA、地址或链路模型、板级可达性和已知限制; 记录官方资料与源码中出现的版本和能力差异。
- Non-goals: 不实现总线或设备驱动; 不为未连接到 CoM260 Kit 的控制器推定板级可用性; 不把 PCI 兼容性并入 MMIO 设备事实。
- Workload: 中到大; 包含五类总线, 需要按统一模板整理并保留各自协议边界。
- Stable baseline: 每类总线都有来源、控制器职责、资源关系、CoM260 可达性和未知项入口。
- Verification boundary: 来源覆盖表中的 I2C、USB、PCIe、CAN 和 EtherCAT 页面均有明确正文落点或可解释的不可访问记录。
- Diagnostic boundary: 资料问题限制在总线类型、控制器实例、DTS、资源依赖、板级连接、版本或来源缺失。
- Split signals: 任一总线形成超过 M02 建议上限的独立资料集时, 在本 milestone 内拆分文档, 不改变其他总线状态。
- Related changes: `establish-k3-peripheral-bus-baseline` (2026-09-12 收尾, archived): 完成 3 篇产品文档 (`docs/buses/k3-i2c-and-usb.md` 127 行 / `docs/buses/k3-pcie-and-can.md` 123 行 / `docs/buses/k3-ethercat.md` 118 行, 均 < 450 行); 70 个唯一 URL 来源覆盖 (在 MS09 70 之上不增减, 5 个总线官网入口由 `deferred` 改为 `active`); 13 个缺口 (G1-G12 + G13, G3/G4/G5 partial 与 G7 open 保持, G13 新增承载总线板级映射 / 软件运行路径 / 恢复闭包); 13 项术语新增 (I2C / USB / DRD / role switch / Hub / PCIe / RC / EP / FlexCAN / CAN-FD / EtherCAT / master / slave); delta spec 同步到 `openspec/specs/k3-peripheral-bus-baseline/spec.md` (7 added, 0 removed, 0 modified); R24 登记; Iteration 000 / Cycle 000 完成 T1 (5 个总线官网入口由 `deferred` 改为 `active` 职责, 复用既有 URL 行, 总数保持 70) + T2 (创建 `k3-i2c-and-usb.md` 127 行, 5 段 (1 范围 / 2 I2C / 3 USB / 4 板级映射 / 5 错误边界与未知项 U1-U4) + 8 个首行 URL 覆盖 + 4 个 U 字段); Iteration 001 / Cycle 000 完成 T3 (创建 `k3-pcie-and-can.md` 123 行, 5 段 (1 范围 / 2 PCIe / 3 CAN / 4 板级映射 / 5 错误边界与未知项 U1-U4) + 7 个首行 URL 覆盖 + 4 个 U 字段, 静态拓扑与运行能力分离); Iteration 002 / Cycle 000 完成 T4 (创建 `k3-ethercat.md` 118 行, 复用 MS07 GMAC 基线, 静态依赖 + 软件/协议边界 + 运行证据缺失); Iteration 003 / Cycle 000 完成 T5-T7 (G13 板级映射 / 运行路径 / 恢复闭包 + 13 项总线术语 + index.md buses 三入口 / 70/53/13 计数 / `docs/buses/` 状态由 `待聚合；R06 状态 deferred` 改为 `已聚合(Iteration 000/001/002)；G7/G13 open`); 7/7 tasks 勾选, 末轮 Plan Review accepted, `Next Cycle: None` / `Next Iteration: None`, Persisted Evidence `none`, 未触动 `others/` 或既有 staged 内容, `git diff --check` 退出 0, `openspec validate --strict` 退出 0。
- Related references: R04, R05, R08, R24

### MS11 — K3 通用外设知识基线

- Status: planned
- Outcome: 形成 GPIO、PWM、IR-RX、Audio、WDT 和 RTC 的可检索、可追溯知识入口。
- Rationale: 这些外围设备已有官网主题但尚未进入正文; 以统一模板整理可提高来源覆盖度, 又避免为每个小主题建立过细 milestone。
- Dependencies: MS03
- Scope: 聚合控制器能力、实例、DTS、clock/reset、pinctrl、IRQ、DMA、板级连接、使用限制和已知缺口; 对 Audio 等涉及 DMA 或跨域资源的主题引用 MS06, 不重复定义所有权模型。
- Non-goals: 不实现外围设备驱动; 不推定 CoM260 未引出的 SoC 控制器可在 Kit 上使用; 不因资料存在而声明功能可用。
- Workload: 中; 六类主题以短文或分节方式整理, 复杂主题通过交叉引用控制重复。
- Stable baseline: 官网已列外围设备均能从总索引进入对应知识说明, 且 SoC 能力与 CoM260 板级可达性分开记录。
- Verification boundary: 来源覆盖表中的 GPIO、PWM、IR-RX、Audio、WDT 和 RTC 页面均有正文落点或明确缺口; 术语和资源引用与既有文档一致。
- Diagnostic boundary: 资料问题限制在设备能力、实例、DTS、clock/reset、pinctrl、IRQ/DMA、板级连接或来源缺失。
- Split signals: Audio 或其他单一主题超过 M02 建议上限时, 在本 milestone 内独立成文。
- Related changes: None
- Related references: R04, R05, R08

### MS12 — K3 镜像制作、烧录与启动操作知识基线

- Status: planned
- Outcome: 形成 K3/CoM260 从 SDK 或第三方 OS 产物，到镜像封装、RAM 引导、介质烧录、启动观察和恢复边界的可追溯流程文档。
- Rationale: MS03 只建立启动和镜像事实，MS09 补齐存储介质与控制器边界；二者都没有系统整理 Titan、Fastboot、SD 卡、FIT/ITB、分区和恢复流程。
- Dependencies: MS03, MS09
- Scope: 聚合 Buildroot、Titan、K3-Ubuntu-Images、U-Boot、OpenSBI、manifests、buildroot-ext 等官方或参考仓库资料；整理 bootinfo、FSBL、ESOS、OpenSBI、U-Boot、bootfs/rootfs 与 payload 的产物关系；区分 FIT 内部 load/entry、上传缓冲地址和介质落点；分别记录 U-Boot RAM 临时引导、Fastboot 分区烧录、Titan ZIP、SD 卡整盘镜像和 local boot；记录 FORCE_RECOVERY/FEL、ADB、U-Boot Shell 等入口的前置状态和适用边界；为每条流程提供前置条件、操作顺序、成功判据、失败分层、停止条件和恢复要求；区分官网事实、官方仓库行为、第三方实现、推论和未确认项；形成正式 `docs/boot/` 主题文档，并同步来源覆盖、术语、缺口和索引。
- Non-goals: 不安装构建或烧录工具；不编译、打包或下载镜像；不连接、复位或操作开发板；不执行 Titan、Fastboot、`mtd erase/write`、`dd`、分区或格式化命令；不写入 UFS、eMMC、SPI-NOR、SD 卡、GPT、bootinfo 或固件分区；不声明任何流程已由本项目真板验证；不把 K3-Ubuntu-Images、Rt-Async-AMP 或其他第三方布局提升为 CoM260 官方规范；不实现 StarryOS 移植、驱动、rootfs 或构建系统。
- Workload: 中到大；需要跨官方文档、多个官方仓库和固定第三方实现对齐产物名、分区、地址、工具入口及恢复语义，并处理当前不可访问页面。
- Stable baseline: 后续规划者可以直接判断目标产物应采用哪类封装、通过哪种部署路径进入哪一启动阶段、哪些动作改变持久状态，以及缺少什么证据时必须停止。
- Verification boundary: 每类产物和部署路径都有来源、适用板型、版本或观察日期；RAM 引导、分区更新、Titan 和 SD 卡路径互不混写；所有命令明确标注来源和“未实跑”状态；破坏性步骤都有前置核对、停止条件和恢复要求；地址、分区、DTS、板型和介质未知项不得由示例补值；文档链接、来源覆盖、术语、缺口与索引一致。
- Diagnostic boundary: 资料问题可限制在产物生成、FIT/ITB 布局、上传与加载地址、分区映射、Fastboot/Titan 入口、SD 启动选择、启动阶段判读或恢复资料缺失，不与存储驱动实现或真板故障混合。
- Split signals: 若 Titan/整盘镜像资料与 U-Boot RAM/Fastboot 路径分别形成可独立维护的大型资料集，或任一正文超过 M02 的 500 行建议上限，则在 MS12 内拆分主题文档；仍保持同一 milestone。
- Related changes: None
- Related references: R05, R07-R09, R12, R18, R20-R22

## 进行中

(无)

## 已承诺待办

(无)

## 阻塞

(无)

## 最近完成

- 项目初始化: 完成 OpenSpec 结构, specs, SNAPSHOT, tasks, change-cycle 模板, CLAUDE.md。
- MS01 / change `establish-k3-doc-foundation` (2026-09-05 收尾, archived): 完成 R01、R04-R08 的 38 个唯一 URL 来源覆盖、主题目录、文档模板、术语种子、6 类已知缺口与人工刷新流程; 修正 `openspec/config.yaml` 的 YAML quoting, 使 OpenSpec CLI 加载既有 artifact rules; 同时建立 MS02 的机制基础 (refresh change 工作流), 但首次真实 source refresh 仍由未来来源变更 change 验收。
- MS02 / change `establish-k3-source-tracking-baseline` (2026-09-05 收尾, archived): 完成 7 URL 首次真实 refresh, 建立持久字段比较与新 baseline 字段二分规则, SPA 壳归 `unreachable`, R07 通过 R08 supporting fallback 取得 `交叉验证` 等级 SDK baseline, 9/9 tasks 勾选, Plan Review accepted-by-explicit-waiver。
- MS03 / change `establish-k3-com260-board-boot-baseline` (2026-09-08 收尾, archived): 完成 4 篇产品文档 (`platform/k3-soc-overview` 179 行 / `platform/com260-board-resources` 303 行 / `boot/com260-boot-chain` 202 行 / `boot/com260-image-and-dts` 210 行); 51 个唯一 URL 来源覆盖 (含 4 个 linux-6.18 raw DTS 文件); 7 个缺口 (G1-G7) 闭包, G3 partial, G7 新增, G8 删除; 直接打开 4 个 DTS 候选文件, CMA 0x140000000 + DRAM 0x102000000 两 cell 解码; 4 份 Rt-Async-AMP 第三方分析已登记 R09-R12; 13/13 tasks 勾选, Plan Review accepted (四轮迭代, 含 5 项 Blocking + 1 Minor + 1 Non-blocking 全部修复, 所有 Gate 5 命令经 shell 实跑)。
- MS06 / change `establish-k3-com260-dma-memory-ownership-baseline` (2026-09-09 收尾, archived): 完成 2 篇产品文档 (`docs/dma/k3-dma-and-memory-ownership.md` 154 行 / `docs/dma/k3-cache-pma-address-translation.md` 79 行, 均 < 450 行); 68 个唯一 URL 来源覆盖 (在 MS05 62 之上新增 6: Iter 000 新增 3 docs-buildroot GitHub 21-DMA / 09-GMAC / ufs + Iter 001 新增 3 RISC-V spec 仓库 riscv-iommu / privileged.adoc / unprivileged.adoc); 10 个缺口 (G1-G10, G3 / G4 / G5 partial, 其余 G1 / G2 / G6 / G7 / G8 / G9 / G10 状态保持); delta spec 同步到 `openspec/specs/k3-com260-dma-memory-ownership-baseline/spec.md`, 登记 R15; Iteration 000 完成 21-DMA / 09-GMAC / ufs docs-buildroot GitHub raw 对应页直接打开 (2026-09-09) + k3-dma-and-memory-ownership.md 创建 (按对象分层: 通用 DMA controller / GMAC 内建 DMA / UFS 内建 DMA / AP↔RP 共享内存 / descriptor 与 data buffer 状态机 / 错误超时取消与恢复 / U1-U4 未知项 4 字段 + 与其他主题文档关系 + 来源与交叉验证导航 8 个首行 URL 覆盖); Iteration 001 完成 k3-cache-pma-address-translation.md 创建 (5 段 cache / PMA / PBMT / IOMMU + 1 段地址空间 + 边界段全部按证据强度标注, 28 处标注, 3 处未知项均含当前证据 / 解除条件) + G5 由 open 调整为 partial (PMA 16 entries + Svpbmt K3 silicon 忽略 + 无 RISC-V IOMMU + `fence iorw,iorw` barrier 行为回写证据) + source-coverage 65 → 68 + index.md URL/缺口计数 62 → 68 / G3,G4 partial → G3,G4,G5 partial / docs/dma/ 行由"待聚合；G5 阻塞 coherency/IOMMU"更新为双 Iteration 标注; 5/5 tasks 勾选, Plan Review accepted-by-explicit-waiver (用户对 Gate 2 User Plan Approval 字段显式豁免; Act 自承担 Plan Review 终态; 风险已记录: 若用户后续要求 Replan, 需把本字段回退至 BLOCKED 并补独立 Plan Review 流程), Persisted Evidence `none`, 未触动 `others/` 或产品代码。
- MS08 / change `establish-k3-amp-rpc-baseline` (2026-09-10 收尾, archived): 完成 2 篇产品文档 (`docs/amp/k3-amp-shared-memory-lifecycle.md` 205 行 / `docs/amp/k3-rpc-ring-notification.md` 246 行, 均 < 450 行); 70 个唯一 URL 来源覆盖 (复用 MS07 70 行, 3 条既有 URL 增加 `amp` 主题职责, 未新增 URL); 11 个缺口 (G1-G10 + G11, G4/G5 partial 与 G7 open 保持, G11 新增承载 ring/RPC 协议闭包); delta spec 同步到 `openspec/specs/k3-amp-rpc-baseline/spec.md` (6 added, 0 removed, 0 modified); 5 项术语新增 (AMP / RPC / ring / doorbell / 共享窗口); 7/7 tasks 勾选, 历经 1 个 initial + 2 个 rework Cycle, 末轮 Plan Review accepted, `Next Cycle: None` / `Next Iteration: None`, Persisted Evidence `none`, 未触动 `others/` 或既有 staged 内容, `git diff --check` 退出 0, `openspec validate --strict` 退出 0。
- MS09 / change `establish-k3-storage-controller-baseline` (2026-09-11 收尾, archived): 完成 2 篇产品文档 (`docs/storage/k3-qspi-spi-sdhci.md` 151 行 / `docs/storage/k3-ufs.md` 153 行, 均 < 450 行); 70 个唯一 URL 来源覆盖 (在 MS08 70 之上不增减, 4 个存储官网入口由 `deferred` 改为 `active`, 复用 1 个 docs-buildroot UFS supporting row, URL 总数保持 70); 12 个缺口 (G1-G11 + G12, G3/G4/G5 partial 与 G7 open 保持, G12 新增承载存储控制器板级映射 / 运行路径 / 恢复闭包); 15 项术语新增 (SPI / QSPI / SDHCI / eMMC / UFS / M-PHY / UniPro / UTP / UPIU / UTRD / UTMRD / UCD / PRDT / SCSI / LUN); delta spec 同步到 `openspec/specs/k3-storage-controller-baseline/spec.md` (7 added, 0 removed, 0 modified); R23 登记; Iteration 000 / Cycle 000 完成 T1 (4 个存储官网入口由 `deferred` 改为 `active` 职责, 复用 1 个 docs-buildroot UFS supporting row, URL 总数保持 70) + T2 (创建 `k3-qspi-spi-sdhci.md` 151 行, 5 段 (1 范围 / 2 QSPI / 3 SPI / 4 SDHCI / 5 错误边界与未知项 U1-U5) + 8 个首行 URL 覆盖 + 5 个 U 字段); Iteration 001 / Cycle 000 完成 T3 (创建 `k3-ufs.md` 153 行, 12 段分层: 1 范围与证据等级 / 2 SoC 能力与板级可达性 / 3 静态资源 / 4 启动关系 / 5 MPHY/UniPro/Link startup / 6 UTP/SCSI descriptor (UTRD/UTMRD/UCD/PRDT/UPIU) / 7 DMA/cache/doorbell / 8 SCSI/LUN scan 与 sync block 边界 / 9 完成模型 (轮询, 不注册 IRQ) / 10 错误超时与恢复 / 11 未知项 U1-U5 / 12 主题边界与导航, 10 个首行 URL 覆盖 + 固定 revision 第三方 `19219411d` 链接 + U1-U5 4 字段); Iteration 002 / Cycle 000 完成 T4-T6 (G12 板级映射 / 运行路径 / 恢复闭包 + 15 项存储术语 + index.md storage 双入口 / 70/40/12 计数 / docs/storage/ 状态由 `待聚合；R06 状态 deferred` 改为 `已聚合(Iteration 000/001)；G5 partial, G7/G12 open`) 后被 Plan Review 标记 rework-required (R1: 10 个 UFS 术语章节定位与数据结构描述错误 / R2: G12 区块 `k3_ufs/transfer.rs` permalink 指向非固定 revision / R3: change `tasks.md` T4-T6 未勾选 / R4: Act 覆盖响应时删除 Plan Review 区域); 同一 Cycle 内 Act Response `reported → pending` 后做有限修复: R1 修正 M-PHY/UniPro → §5 / UTP/UPIU/UTRD/UTMRD/UCD/PRDT → §6 / SCSI/LUN → §8 章节定位与语义 (UTRD 改 command type / data direction / interrupt bit / OCS / UCD base / response UPIU offset+length / PRDT offset+length; UTMRD 改 "只分配并编程, 未观察到 task-management 提交"; UCD 改 "command UPIU + response UPIU 各 512-byte 对齐区域"; PRDT 移除 DBC 20 bit 第三方数值; UPIU 移除 OCS/response 错误归属; SCSI 列表移除 NOP/QUERY 错误项); R2 把 G12 区块 permalink 从 `github.com/rt-async-amp/tgoskits` 替换为 `github.com/PlaticaIt/StarryOS/blob/19219411d.../k3_ufs/transfer.rs` 与 `k3-ufs.md:68,100` 同款; R3 把 change `tasks.md` 第 6-8 行 `[ ]` 改为 `[x]`; R4 覆盖 Act Response 时未触动 `## Plan Review` 区域; 6/6 tasks 勾选, 末轮 Plan Review accepted, `Next Cycle: None` / `Next Iteration: None`, Persisted Evidence `none`, 未触动 `others/` 或既有 staged 内容, `git diff --check` 退出 0, `openspec validate --strict` 退出 0。
- MS10 / change `establish-k3-peripheral-bus-baseline` (2026-09-12 收尾, archived): 完成 3 篇产品文档 (`docs/buses/k3-i2c-and-usb.md` 127 行 / `docs/buses/k3-pcie-and-can.md` 123 行 / `docs/buses/k3-ethercat.md` 118 行, 均 < 450 行); 70 个唯一 URL 来源覆盖 (在 MS09 70 之上不增减, 5 个总线官网入口由 `deferred` 改为 `active`); 13 个缺口 (G1-G12 + G13, G3/G4/G5 partial 与 G7 open 保持, G13 新增承载总线板级映射 / 软件运行路径 / 恢复闭包); 13 项术语新增 (I2C / USB / DRD / role switch / Hub / PCIe / RC / EP / FlexCAN / CAN-FD / EtherCAT / master / slave); delta spec 同步到 `openspec/specs/k3-peripheral-bus-baseline/spec.md` (7 added, 0 removed, 0 modified); R24 登记; Iteration 000 / Cycle 000 完成 T1 (5 个总线官网入口由 `deferred` 改为 `active` 职责, 复用既有 URL 行, 总数保持 70) + T2 (创建 `k3-i2c-and-usb.md` 127 行, 5 段 (1 范围 / 2 I2C / 3 USB / 4 板级映射 / 5 错误边界与未知项 U1-U4) + 8 个首行 URL 覆盖 + 4 个 U 字段, 区分 I2C controller/client 与 USB controller/PHY/Host/DRD/role switch/Hub 共享 PHY); Iteration 001 / Cycle 000 完成 T3 (创建 `k3-pcie-and-can.md` 123 行, 5 段 (1 范围 / 2 PCIe / 3 CAN / 4 板级映射 / 5 错误边界与未知项 U1-U4) + 7 个首行 URL 覆盖 + 4 个 U 字段, 静态拓扑与运行能力分离, AP/RP FlexCAN 三层分离 + CAN C2 板级冲突保留); Iteration 002 / Cycle 000 完成 T4 (创建 `k3-ethercat.md` 118 行, 复用 MS07 GMAC 基线 + 静态依赖 `ec_master → master0 → main-device = <&eth1>` + 软件/协议边界 + 周期/同步/恢复/真板日志均标为未知); Iteration 003 / Cycle 000 完成 T5-T7 (G13 板级映射 / 运行路径 / 恢复闭包 + 13 项总线术语 + index.md buses 三入口 / 70/53/13 计数 / `docs/buses/` 状态由 `待聚合；R06 状态 deferred` 改为 `已聚合(Iteration 000/001/002)；G7/G13 open`); 7/7 tasks 勾选, 末轮 Plan Review accepted, `Next Cycle: None` / `Next Iteration: None`, Persisted Evidence `none`, 未触动 `others/` 或既有 staged 内容, `git diff --check` 退出 0, `openspec validate --strict` 退出 0。

## 与 OpenSpec Changes 的同步

- 每个 milestone 对应一个或多个 OpenSpec change (数量不绑定)。
- change `accepted` 时, 由 `openspec-docs-maintainer` 按完成范围把对应 milestone 推进到 `active` 或 `completed`, 并在"最近完成"追加引用。
- 未批准的想法不进入 tasks。
