# SNAPSHOT

> 当前项目状态: `current`
> 最后同步: 2026-09-10 (Thu Sep 10 2026 17:05 GMT+0800) 增量刷新
> 同步者: `openspec-docs-maintainer`
> 同步 revision: `b484827` + 未提交 change `establish-k3-com260-dma-memory-ownership-baseline` + change `establish-k3-com260-gmac-network-baseline` + change `establish-k3-amp-rpc-baseline` 三 change 收尾同步 (MS06 + MS07 + MS08 全部产品交付 + 项目级状态; 在 MS05 同步基线之上增量刷新)

## 项目身份

- 名称: K3 板子信息汇总
- 英文名: SpacemiT K3 Board Documentation Aggregation
- 用途: 把 SpacemiT 官方社区的 K3 板子文档整理为结构化 Markdown, 便于离线检索与跨会话复用。
- 范围: 仅 K3 (key_stone/k3); 不覆盖 SpacemiT 其它板子或 K3 之外的章节。

## 信息源 (单源, M01)

- URL: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs
- 语言: 简体中文
- 权威性: Spacemit 官方社区文档
- 最近观察修订日期: 2026-09-08 (initial observation baseline 2026-09-02 由 change `establish-k3-doc-foundation` 记录; 2026-09-05 由 refresh change `establish-k3-source-tracking-baseline` 更新; 2026-09-08 由 change `establish-k3-com260-board-boot-baseline` 在 §6.6 重新观察确认; 2026-09-08 由 change `establish-k3-com260-platform-uart-baseline` 在 §3.2 / §6 重新观察确认; R01 仍为 SPA 壳, 实际可见身份为 Vue SPA title="SpacemiT", `partially-observed` 状态保持)

## 形态与目录

- 类型: 纯文档 / Markdown 聚合
- 仓库内不存放可执行代码 (M04)
- 目录:
  - `docs/` — K3 聚合产出 (主题驱动, 非镜像源页面树, D02)
  - `docs/index.md` — 总入口与主题职责表
  - `docs/reference/` — 来源覆盖、文档模板、术语、已知缺口与人工刷新指南
  - `docs/boot/` — K3 启动链、镜像与 DTS 主题文档
  - `docs/platform/` — K3 SoC 概述、CoM260 板级资源与平台控制 (pinctrl/clock/reset/APBC/CCU) 主题文档
  - `docs/serial/` — K3 UART 控制器实例、CoM260 UART0 物理接口与 console 链路主题文档
  - `docs/amp/` — K3 AP/RP 生命周期、共享窗口、ring、RPC 与通知机制主题文档
  - `openspec/` — OpenSpec 配置, specs, changes (active 与 archive)
  - `.claude/` — Claude Code 入口 (skills, commands, docs, analysis, runbooks, incidents)
  - `CLAUDE.md` — 项目公共规则
  - `AGENTS.md` — 不适用 (本项目仅支持 Claude Code)

## 技术栈与依赖

- 无运行时依赖
- 无构建工具链
- 无包管理文件
- 唯一工具: OpenSpec CLI 1.6.0 (用于变更管理和规范验证)

## 支持范围

- Claude Code: 已配置 (`.claude/skills/`, `.claude/commands/`)
- Codex: 已配置 (`.agents/skills/` 软链, `AGENTS.md` 入口)
- OpenCode: 已配置 (`.agents/skills/` 软链, `AGENTS.md` 入口)
- skill 副本: 单一权威在 `.claude/skills/`, `.agents/skills/` 为软链, 不维护多份
- skill frontmatter: 统一精简为 `name` + `description` 两字段 (三端共同)

## 工作区与分支

- 工作区: change `establish-k3-doc-foundation` / `establish-k3-source-tracking-baseline` / `establish-k3-com260-board-boot-baseline` / `establish-k3-com260-platform-uart-baseline` / `establish-k3-com260-interrupt-time-notification-baseline` / `establish-k3-com260-dma-memory-ownership-baseline` / `establish-k3-com260-gmac-network-baseline` / `establish-k3-amp-rpc-baseline` 已分别于 2026-09-05 / 2026-09-05 / 2026-09-08 / 2026-09-08 / 2026-09-09 / 2026-09-09 / 2026-09-10 / 2026-09-10 收尾归档 (MS01 / MS02 / MS03 / MS04 / MS05 / MS06 / MS07 / MS08); 工作区待常规 commit 的产品代码包括 `docs/index.md` / `docs/boot/com260-boot-chain.md` / `docs/boot/com260-image-and-dts.md` / `docs/platform/com260-board-resources.md` / `docs/platform/k3-soc-overview.md` / `docs/platform/k3-platform-control.md` / `docs/serial/com260-uart.md` / `docs/interrupts/k3-interrupt-and-time.md` / `docs/interrupts/com260-mailbox-notification.md` / `docs/dma/k3-dma-and-memory-ownership.md` / `docs/dma/k3-cache-pma-address-translation.md` / `docs/network/com260-gmac-phy.md` / `docs/network/k3-gmac-dma-irq.md` / `docs/amp/k3-amp-shared-memory-lifecycle.md` / `docs/amp/k3-rpc-ring-notification.md` / `docs/reference/source-coverage.md` / `docs/reference/known-gaps.md` / `docs/reference/source-refresh.md` / `docs/reference/terminology.md` 与 `.claude/analysis/rt-async-amp-k3-{boot-platform,drivers,shared-memory,starryos-reuse}.md`。
- Git 分支: `main`
- 归档目录: `openspec/changes/archive/2026-09-05-establish-k3-doc-foundation/`, `openspec/changes/archive/2026-09-05-establish-k3-source-tracking-baseline/`, `openspec/changes/archive/2026-09-05-establish-k3-com260-board-boot-baseline/`, `openspec/changes/archive/2026-09-08-establish-k3-com260-platform-uart-baseline/`, `openspec/changes/archive/2026-09-09-establish-k3-com260-interrupt-time-notification-baseline/`, `openspec/changes/archive/2026-09-09-establish-k3-com260-dma-memory-ownership-baseline/`, `openspec/changes/archive/2026-09-10-establish-k3-com260-gmac-network-baseline/`, `openspec/changes/archive/2026-09-10-establish-k3-amp-rpc-baseline/`

## 已建立文档

- `docs/index.md`: 总入口、CoM260 范围、九个 reference / boot / platform 链接与十类主题职责 (T2, T8, MS03 T13)
- `docs/reference/source-coverage.md`: 70 个唯一 URL 唯一覆盖表 (MS01-MS02 baseline 38 + MS03 Iter 000 新增 6 + Iter 001 新增 7 + MS04 Iter 000 新增 8 + MS05 Iter 000 新增 3 binding + MS06 Iter 000 新增 3 docs-buildroot GitHub 21-DMA / 09-GMAC / ufs + Iter 001 新增 3 RISC-V spec 仓库 riscv-iommu / privileged.adoc / unprivileged.adoc + MS07 Iter 001 新增 1 K3 GMAC glue driver `dwmac-spacemit-ethqos.c`), R01/R04-R08、linux-6.18 仓库 `k3-br-v1.0.y` 分支 raw DTS / binding / driver URL、docs-buildroot 05-UART.md / 21-DMA.md / 09-GMAC.md / ufs.md / com260_ds.md 等 8 个 MS04 新增 URL、3 个 MS06 docs-buildroot URL 与 1 个 MS07 K3 GMAC glue driver 已逐项登记 (T3, MS03 T7-T9, MS04 T2, MS06 T3, MS07 T3)
- `docs/reference/document-template.md`: 首行来源、源端修订、观察日期与证据强度格式 (T4)
- `docs/reference/terminology.md`: K3、CoM260 Kit、SoC、AP、RCPU、AIA、APLIC、IMSIC、MMIO、IRQ、DMA、IOMMU、GMAC、PHY、MDIO、RGMII、polling、async、waker、coherency 二十项主写法与别名 (T5)
- `docs/reference/known-gaps.md`: 10 个缺口 (G1-G10) 含 G3 / G4 / G5 partial, MS03 Iter 001 / T11 新增 G7 (CoM260 Kit 默认目标 DTS 未唯一映射), MS04 Iter 001 / T3 新增 G8 (K3 SoC `uart10` base 偏移)、G9 (`spacemit,k1-uart` compatible 字符串的 K3 硬件边界)、G10 (K3 UART 完整寄存器语义与电气映射), MS05 Iter 001 / T3 把 G4 由 `open` 调整为 `partial` (AP/RP wired IRQ + MSI 路径有官方 k3.dtsi / binding 静态拓扑证据, 缺 AP 域真板 MSI delivery 与 IMSIC EID), MS06 Iter 001 / T3 把 G5 由 `open` 调整为 `partial` (PMA 16 entries + Svpbmt K3 silicon 忽略 + 无 RISC-V IOMMU + `fence iorw,iorw` barrier 行为有 OpenSBI `7a2df08` + Rt-Async-AMP `ccb1ff0b` 第三方源码交叉验证, 缺 K3 cache line 大小 / PMA mode 编码全集 / IOMMU 节点细节); 状态 7 `open` + 3 `partial`, 源端 `docs-product/com260_ds.md` / `k3.dtsi` / `k3-rdomain.dtsi` / `8250.yaml` / `8250_of.c` / `spacemit_k3.c` / `chip-k3-rt24/src/lib.rs` / `ov-shm/src/shm.rs` 已逐项登记 (T6, MS03 T12, MS04 T5, MS05 T4, MS06 T4)
- `docs/reference/source-refresh.md`: 持久字段 / 新 baseline 字段 / SPA 壳 / 真正的停止条件 四节, 以及 MS02 建立的 refresh change 工作流 (T7, MS02)
- `docs/platform/k3-soc-overview.md`: K3 SoC 能力概述, 涵盖 CPU/内存/外设/连接/启动能力总览 (MS03 Iteration 000 交付)
- `docs/platform/com260-board-resources.md`: K3 CoM260 模组 + Kit 板级资源, 涵盖 SoC 集成接口、模组引脚、Kit 底板连接器、PHY、UART、debug 等 (MS03 Iteration 000 交付)
- `docs/platform/k3-platform-control.md`: K3 SoC 平台控制资源按 AP / APBC2 secure / RCPU 三域分离的 pinctrl / clock / reset / APBC / CCU provider 依赖与 UART consumer 字段映射; 包含 8.6 节点 AP 域 `uart10` base 偏移未知项, 与 `com260-uart.md` §10.1 / `known-gaps.md` G8 交叉引用 (MS04 Iteration 000 交付, 313 行)
- `docs/boot/com260-boot-chain.md`: K3 SoC 启动能力与 K3 CoM260 Kit 已观察到的启动链路 (local boot: Boot ROM → FSBL/SPL → ESOS → OpenSBI → U-Boot → payload/OS; download boot: Boot ROM → U-Boot Fastboot), 介质、SDK 版本边界、未知闭包 (MS03 T10, 202 行)
- `docs/boot/com260-image-and-dts.md`: K3 CoM260 镜像类型、写入方式、DTS 候选集合 (6 个 .dts + 1 个 .dtsi base, 直接打开 4 个文件), CMA 0x140000000 + DRAM 0x102000000 解码, 缺口闭包 (MS03 T11, 210 行)
- `docs/serial/com260-uart.md`: K3 SoC 17 个 UART 物理实例 (AP 域 10 + APBC2 secure 1 + RCPU 域 6) 的 DTS 字段、CoM260 UART0 物理接口到 `uart0` 节点与静态 console 链路、来源冲突 (FIFO 深度 256 vs 64) 与固定 revision 第三方 (Rt-Async-AMP/tgoskits PXA UART) 经验; 17 实例矩阵、aliases 映射、`uart10` base 偏移未知项闭包; 首行覆盖 10 个官方直接 URL, §8 第三方外链转本地相对路径 (MS04 Iteration 001 交付, 316 行)
- `docs/interrupts/k3-interrupt-and-time.md`: K3 AP/RP 中断域分域事实包, 记录 AP CLINT `0xe081c000/0x4000` + 16 hart software/timer interrupts、IMSIC `0xe0400000/0x400000` + 511 IDs / 63 guest IDs / 4/6 index bits、APLIC `0xe0804000/0x4000` + MSI parent IMSIC + 512 wired sources 等官方 `k3.dtsi` 静态拓扑, 第三方 R10-R12 固定 revision 行为单列; U1-U5 未知项各 4 字段, 6 个首行来源元数据与覆盖表精确一致 (MS05 Iteration 000 交付, 152 行)
- `docs/interrupts/com260-mailbox-notification.md`: K3 CoM260 AP↔RP mailbox 双向通知链, 区分物理 mailbox4 与历史变量 `MBX3` 命名空间, AP→RP mailbox4 ch0 / PLIC source 69、RP→AP ch1 / APLIC source 217, 初始化 drain/clear/enable, ISR FIFO→pending→RAW&EN 重读, AP 32×64 drain 上限, 单槽 waker 与多等待者覆盖风险, 第三方 boot/ioctl self-test 3×100000 spin 的能力与未运行边界; IMSIC EID 保留为未知项; 首行 3 来源、U1-U4 未知项各 4 字段 (MS05 Iteration 001 交付, 187 行)
- `docs/dma/k3-dma-and-memory-ownership.md`: K3 DMA 控制器 (三类传输对象: 通用 DMA controller / GMAC 内建 DMA / UFS 内建 DMA) 的描述、AP↔RP 共享内存 + mailbox 通知 + descriptor / data buffer 状态机、错误超时取消与恢复、未知项 U1-U4 各 4 字段; 首行 8 个官方直接 URL, 与 `k3-cache-pma-address-translation.md` §1-§6 + `known-gaps.md` U1-U4 / G5 / G7 交叉引用 (MS06 Iteration 000 交付, 154 行)
- `docs/dma/k3-cache-pma-address-translation.md`: K3 SoC 的 cache 行为 (L1 bypass / L2 预取 / CVA6 限制) + PMA 机制 (16 entries / Svpbmt K3 silicon 忽略 / AMP window 翻转) + PBMT (Svpbmt PTE 扩展 + K3 上 no-op) + IOMMU (K3 当前证据不含 + 替代机制 mailbox + fence + PMA) + 地址空间 (AMP window + RCPU 本地别名窗口) 五维综合主题; 7 段 (1 范围 / 2 地址空间 / 3 Cache / 4 PMA / 5 PBMT / 6 IOMMU / 7 边界), 28 处证据强度标注, 3 处未知项均含当前证据 / 解除条件; 首行 3 RISC-V spec 官方仓库 URL (riscv-iommu / privileged.adoc / unprivileged.adoc), 与 `k3-dma-and-memory-ownership.md` U1-U4 / G5 / G7 交叉引用, 与 `source-coverage.md` 3 行 RISC-V spec 仓库一一对应登记 (MS06 Iteration 001 交付, 103 行)
- `docs/network/com260-gmac-phy.md`: K3 SoC GMAC 能力 (4 路支持 RGMII/RMII/MII 和 TSN) → CoM260 模组 PHY1/GMAC1 引出 → CoM260 DTS 变体 (`k3_com260.dts` / `k3_com260.dtsi` 共享 / `k3_com260_kit_v02.dts`) `&eth1` 节点 → MMIO `0xcac82000/0x2000` (固定 revision 第三方 IFX DTS 唯一可审计) + AP APLIC source 133 (同) → MDIO Clause 22 + PHY address 1 + RGMII phase 47/53 + reset GPIO/delay; 5 段 (1 范围 / 2 SoC 与模组 / 3 板级与 DTS 变体 / 4 platform resources / 5 MDIO/PHY/RGMII/reset/delay) + 5 个四字段未知项 (PHY 型号 / 完整寄存器 / 运行时 link / 实物型号 / 真板 IRQ delivery); 首行 9 个官方直接 URL, 与 `k3-gmac-dma-irq.md` §1-§11 + `k3-dma-and-memory-ownership.md` + `k3-interrupt-and-time.md` + `k3-platform-control.md` 交叉引用, 明确不指定默认 Kit DTS、不由 PHY ID 推定完整器件 (MS07 Iteration 000 交付, 238 行)
- `docs/network/k3-gmac-dma-irq.md`: DWMAC5 MAC / MTL / DMA 三层与 TX/RX descriptor / data buffer ownership 状态链 (CPU prepare → OWN → cache clean → tail / ST / SR doorbell → DMA in-flight → IRQ 或主动 reclaim → W1C / invalidate / completion check → CPU 回收); cache/地址宽度/fence 边界; IRQ cause / mask / W1C / reclaim 路径与 `try_lock` 锁竞争下的潜在推进风险; 设备专有错误 (descriptor error、TBU/RBU、FBE、DMA soft reset timeout、U-Boot 残留 DMA 状态、link-down) 的已证行为与未知恢复边界; 4 个四字段未知项 (K3 descriptor/IRQ programmer reference / GMAC cache 与 device address 模型 / `try_lock` 后确定性推进 / Fatal error 与 reset-link 恢复); 首行 4 个官方直接 URL (SpacemiT 09-GMAC + 官方 GitHub docs-buildroot / linux-6.18 `dwmac-spacemit-ethqos.c` glue driver + `k3_com260_kit_v02.dts`), 固定 revision 第三方证据边界为 Rt-Async-AMP `ccb1ff0b` / tgoskits `19219411d` (`feat/rt-async-amp`); 与 `com260-gmac-phy.md` + `k3-dma-and-memory-ownership.md` + `k3-cache-pma-address-translation.md` + `k3-interrupt-and-time.md` + `k3-platform-control.md` + `known-gaps.md` G3–G7 交叉引用 (MS07 Iteration 001 交付, 206 行)
- `docs/amp/k3-amp-shared-memory-lifecycle.md`: K3 AP/RP 镜像/握手 → 启动链破坏者（SPL/U-Boot/bootm 写共享 SRAM）→ AP reserved-memory + OpenSBI PMA 前置 → AP probe 保留 valid 或初始化 invalid 窗口 → RP wait_ready 观察 magic 并发布 SHM_BASE → 双端在线 → magic 无效或对端 reset → watchdog/fallback re-init → 未读 ring 状态可能清除的完整生命周期模型; AP 物理地址 `0xc0800000` 与 RP 本地 alias 0（同一 SRAM 关系未证）/ window `0x19000` / 初始化竞争 / PMA/PBMT/cache 边界 / reset/re-init 状态转换 / 5 个四字段未知项 (地址 alias 真板证据 / `0` 起点真板可见性 / `fallback_ms` 与 `magic_watchdog` 时序对 K3 适用性 / `SpShmOwner` 调用约束与可达性 / `RtShmDevice::new` 真板初始化者分配); 首行 3 个官方直接 URL (boot.md + image.md + linux-6.18 `k3.dtsi`), 与 `k3-rpc-ring-notification.md` + `k3-dma-and-memory-ownership.md` + `k3-cache-pma-address-translation.md` + `com260-mailbox-notification.md` + `k3-platform-control.md` + `com260-boot-chain.md` + `com260-image-and-dts.md` + `known-gaps.md` G4/G5/G7/G11 交叉引用 (MS08 Iteration 000 交付, 205 行)
- `docs/amp/k3-rpc-ring-notification.md`: K3 共享 ring + mailbox 通知 + 等待者模型分层的 RPC 知识包, 区分三条共享通道 (request/response/urgent) 与 mailbox hardware channel; 端到端发布路径 (write → SeqCst fence → 读 BUSY → 条件 NOTIFY → ISR/FIFO/RAW&EN 重检 → response 关联); 弹性轮询窗口与 BUSY=0 条件 NOTIFY 边界; 错误/取消/超时/reset/restore 状态机; 4 个四字段未知项 (K3 ring 原子序 / 等待者不均与 waker 风险 / 多块发布与 capacity / reset 恢复未读消息); 首行 3 个官方直接 URL, 与 `k3-amp-shared-memory-lifecycle.md` + `k3-dma-and-memory-ownership.md` + `k3-cache-pma-address-translation.md` + `com260-mailbox-notification.md` + `k3-interrupt-and-time.md` + `k3-platform-control.md` + `com260-boot-chain.md` + `com260-uart.md` + `known-gaps.md` G4/G5/G7/G11 交叉引用 (MS08 Iteration 001 交付, 246 行, 经 002-rework 收尾 BUSY 仅描述为弹性轮询状态提示, NOTIFY 明确为 BUSY=0 条件动作)

## 已建立分析

- `.claude/analysis/k3-official-docs-for-starryos-async-drivers.md`: K3 官方资料规模下界与首批聚合主题分析 (R03, MS01)
- `.claude/analysis/rt-async-amp-k3-boot-platform.md`: Rt-Async-AMP 的 K3 启动与板级适配分析 (R09, MS03-MS08)
- `.claude/analysis/rt-async-amp-k3-shared-memory.md`: Rt-Async-AMP 的 K3 共享内存与通知链分析 (R10, MS05/MS06/MS08)
- `.claude/analysis/rt-async-amp-k3-drivers.md`: Rt-Async-AMP 的 K3 驱动与 StarryOS 异步边界分析 (R11, MS04-MS07)
- `.claude/analysis/rt-async-amp-k3-starryos-reuse.md`: Rt-Async-AMP 面向 StarryOS 的 K3 复用清单 (R12, MS03-MS11)

## Milestone 状态指针

- MS01 (来源覆盖与主题结构基线) — `completed` (2026-09-05 收尾, 见 `.claude/docs/tasks.md`)
- MS02 (来源追踪与人工刷新基线) — `completed` (2026-09-05 收尾, change `establish-k3-source-tracking-baseline` 首次真实 refresh 完成 7 URL 逐 URL 结论与 `交叉验证` 等级 SDK baseline)
- MS03 (K3 CoM260 板级与启动事实基线) — `completed` (2026-09-08 收尾, change `establish-k3-com260-board-boot-baseline` 完成 platform/boot 主题文档 + 51 URL 覆盖 + 7 个缺口闭包 + 4 个 raw DTS 文件直接打开)
- MS04 (CoM260 平台控制与串口知识基线) — `completed` (2026-09-08 收尾, change `establish-k3-com260-platform-uart-baseline` 完成 2 篇产品文档 (k3-platform-control 313 行 / com260-uart 316 行) + 59 URL 覆盖 + 10 个缺口 (G3 partial) + 17 实例 UART 矩阵 + 3 个 raw DTS 直接打开 + delta spec 同步 R13; Iter 001 / Cycle 001-rework 修复 AP 域地址范围离散集 (T3-R1)、G8 节点集合 (T5-R1)、§8.6 跨文档一致 (T6-R1))
- MS05 (CoM260 中断、时间与通知机制知识基线) — `completed` (2026-09-09 收尾, change `establish-k3-com260-interrupt-time-notification-baseline` 完成 2 篇产品文档 (k3-interrupt-and-time 152 行 / com260-mailbox-notification 187 行) + 62 URL 覆盖 (在 MS04 59 之上新增 3 binding) + 10 个缺口 (G4 由 open 调整为 partial, 其余 G1-G3/G5-G10 状态保持) + delta spec 同步 R14; Iteration 000 / Cycle 000 修复 6 项 blocking (F1-F6: AP APLIC 拓扑结论、IMSIC num-ids、RP UART IRQ 范围、§3 拓扑观察、§4 时间结论、Timer.md/k3.dtsi 观察日期撤销), Iteration 001 / Cycle 000 在同 Cycle 有限返修关闭单一证据语义 blocker; 5/5 tasks 勾选, 两轮 Plan Review 均 accepted)
- MS06 (CoM260 DMA、cache、PMA 与内存所有权知识基线) — `completed` (2026-09-09 收尾, change `establish-k3-com260-dma-memory-ownership-baseline` 完成 2 篇产品文档 (k3-dma-and-memory-ownership 154 行 / k3-cache-pma-address-translation 79 行) + 68 URL 覆盖 (在 MS05 62 之上新增 3 docs-buildroot 21-DMA/09-GMAC/ufs + 3 RISC-V spec 仓库) + 10 个缺口 (G5 由 open 调整为 partial, 其余 G1-G4/G6-G10 状态保持) + delta spec 同步 R15; Iteration 000 完成 docs-buildroot 21-DMA / 09-GMAC / ufs raw 对应页直接打开 (2026-09-09) + k3-dma-and-memory-ownership.md 创建 (按对象分层: 通用 DMA controller / GMAC 内建 DMA / UFS 内建 DMA / AP↔RP 共享内存 / descriptor 与 data buffer 状态机 / 错误超时取消与恢复 / U1-U4 未知项 4 字段 + 与其他主题文档关系 + 来源与交叉验证导航 8 个首行 URL 覆盖); Iteration 001 完成 k3-cache-pma-address-translation.md 创建 (5 段 cache / PMA / PBMT / IOMMU + 1 段地址空间 + 边界段全部按证据强度标注, 28 处标注, 3 处未知项均含当前证据 / 解除条件) + G5 由 open 调整为 partial (PMA 16 entries + Svpbmt K3 silicon 忽略 + 无 RISC-V IOMMU + `fence iorw,iorw` barrier 行为回写证据) + source-coverage 65 → 68 + index.md URL/缺口计数 62 → 68 / G3,G4 partial → G3,G4,G5 partial / docs/dma/ 行由"待聚合；G5 阻塞 coherency/IOMMU"更新为双 Iteration 标注; 5/5 tasks 勾选, Plan Review accepted-by-explicit-waiver (用户对 Gate 2 User Plan Approval 字段显式豁免; Act 自承担 Plan Review 终态; 风险已记录: 若用户后续要求 Replan, 需把本字段回退至 BLOCKED 并补独立 Plan Review 流程), Persisted Evidence `none`, 未触动 `others/` 或产品代码)
- MS07 (CoM260 GMAC、MDIO、PHY 与网络硬件知识基线) — `completed` (2026-09-10 收尾, change `establish-k3-com260-gmac-network-baseline` 完成 2 篇产品文档 (`docs/network/com260-gmac-phy.md` 238 行 / `docs/network/k3-gmac-dma-irq.md` 206 行, 均 < 450 行) + 70 URL 覆盖 (在 MS06 68 之上新增 K3 GMAC glue driver `dwmac-spacemit-ethqos.c`, URL 总数 69 → 70) + 10 个缺口 (G3 / G4 / G5 = `partial`, G6 / G7 = `open`, G1-G2 / G8-G10 状态保持) + delta spec 同步到 `openspec/specs/k3-com260-gmac-network-baseline/spec.md`（5 added, 0 removed, 0 modified） + R16 登记; Iter 000 完成 `com260-gmac-phy.md` 创建 (SoC/模组/Kit/DTS 变体 + `eth1` 平台资源 + MDIO/PHY/RGMII 静态链 + 来源差异/运行时边界 + 5 个四字段未知项), Iter 000 / Cycle 001-rework 修复 PLAN-INVALID（来源任务越界） + ACT-DEVIATION（§4.1 MMIO/IRQ 证据等级越级） 2 项 Important 收敛 accepted; Iter 001 完成 `k3-gmac-dma-irq.md` 创建 (MAC/MTL/DMA 分层 + TX/RX 状态机 + descriptor/buffer 分离 + cache/doorbell + IRQ/W1C/reclaim + 错误恢复 + `try_lock` 推进风险 + 4 个四字段未知项) + G3–G7 收敛 + index 接入 (T5 修正 G4 汇总 `open → partial` 与 k3.dtsi 静态拓扑一致) + §8 DMA soft reset timeout (Acceptance 3: `core 初始化返回错误` → `记录 warning 后继续 stop DMA / 重配 / 启动 DMA / 返回 Ok(())`) + U4 同步 reset → warning → 继续初始化语义 + index 维护规则计数 69 → 70 (Acceptance 5) 2 项 Plan Review Important 修复 (Plan Review 复审 accepted); 6/6 tasks 勾选, Persisted Evidence `none`, 未触动 `others/` 或既有 staged 内容)
- MS08 (K3 AMP、共享内存、RPC 与跨核通信知识基线) — `completed` (2026-09-10 收尾, change `establish-k3-amp-rpc-baseline` 完成 2 篇产品文档 (`docs/amp/k3-amp-shared-memory-lifecycle.md` 205 行 / `docs/amp/k3-rpc-ring-notification.md` 246 行, 均 < 450 行) + 70 URL 来源覆盖 (复用 MS07 70 行, 3 条既有 URL 增加 `amp` 主题职责, 未新增 URL) + 11 个缺口 (G1-G10 + G11, G4/G5 partial 与 G7 open 保持, G11 新增承载 ring/RPC 协议闭包) + delta spec 同步到 `openspec/specs/k3-amp-rpc-baseline/spec.md` (6 added, 0 removed, 0 modified) + R17 登记; Iter 000 / Cycle 000 完成 T1 (3 条既有官方 URL 增加 `amp` 主题职责) + T2 (创建 `k3-amp-shared-memory-lifecycle.md`, 5 个四字段未知项, 11 个相对链接全部有效), Plan Review 接受的字段说明补 `amp` 同样在当前 Cycle 内部完成; Iter 001 / Cycle 000 完成 T3-T7 (创建 `k3-rpc-ring-notification.md` 246 行 + G11 + 5 项术语 + index AMP 入口与计数) 后被 Plan Review 标记 `rework-required` (Acceptance 2 BUSY=1 行为错误 + Act Response 重复); Iter 001 / Cycle 001-rework 修复 T4-R1 (BUSY=1 描述为"服务端处于可发现请求的弹性窗口") + T4-R2 (父 Cycle Act Response 单一化); Iter 001 / Cycle 002-rework 修复 T4-R3 (三处 BUSY/NOTIFY 表述越界, 全文 BUSY 仅描述为弹性轮询状态提示, NOTIFY 明确为 BUSY=0 条件动作, 不再声称并发保护, 全文 BUSY 扫描无锁/互斥保证或无条件 NOTIFY 相反表述); 7/7 tasks 勾选, 末轮 Plan Review accepted, `Next Cycle: None` / `Next Iteration: None`, Persisted Evidence `none`, 未触动 `others/` 或既有 staged 内容, `git diff --check` 退出 0, `openspec validate --strict` 退出 0)
- MS09-MS11 — `planned`

## 关键约束摘要 (指针)

- M01 单一权威源 — `openspec/specs/project-model/spec.md`
- M02 输出形态 — `openspec/specs/project-model/spec.md`
- M03 语言与术语 — `openspec/specs/project-model/spec.md`
- M04 不存放可执行代码 — `openspec/specs/project-model/spec.md`
- D01 纯 Markdown 聚合 — `openspec/specs/decisions/spec.md`
- D02 主题驱动目录 — `openspec/specs/decisions/spec.md`
- D03 六个实际文档承载基础职责 — `openspec/specs/decisions/spec.md`
- D04 URL 是覆盖记录的唯一键 — `openspec/specs/decisions/spec.md`
- D05 主题文档首行同时表达修订与观察时间 — `openspec/specs/decisions/spec.md`
- D06 证据强度采用四级标记 — `openspec/specs/decisions/spec.md`
- D07 刷新状态与聚合状态分离 — `openspec/specs/decisions/spec.md`
- D08 配置修复只增加必要引用 — `openspec/specs/decisions/spec.md`
- R01 权威源 URL — `openspec/specs/references/spec.md`
- R03-R12 当前有用、持续观察、第三方分析 URL/文档集合 — `openspec/specs/references/spec.md`
- R13 MS04 delta spec (`openspec/specs/k3-com260-platform-uart-baseline/spec.md`) — `openspec/specs/references/spec.md`
- R14 MS05 delta spec (`openspec/specs/k3-com260-interrupt-time-notification-baseline/spec.md`) — `openspec/specs/references/spec.md`
- R15 MS06 delta spec (`openspec/specs/k3-com260-dma-memory-ownership-baseline/spec.md`) — `openspec/specs/references/spec.md`
- R16 MS07 delta spec (`openspec/specs/k3-com260-gmac-network-baseline/spec.md`) — `openspec/specs/references/spec.md`
- R17 MS08 delta spec (`openspec/specs/k3-amp-rpc-baseline/spec.md`) — `openspec/specs/references/spec.md`

## 同步状态

- `current`: 本次增量刷新同步了 change `establish-k3-amp-rpc-baseline` 收尾后的项目状态; MS08 已完成, 2 篇产品文档 (`docs/amp/k3-amp-shared-memory-lifecycle.md` 205 行 / `docs/amp/k3-rpc-ring-notification.md` 246 行, 均 < 450 行) + 70 URL 来源覆盖 (复用 MS07 70 行, 3 条既有 URL 增加 `amp` 主题职责, 未新增 URL) + 11 个缺口 (G1-G10 + G11, G4/G5 partial 与 G7 open 保持, G11 新增) + delta spec 同步到 `openspec/specs/k3-amp-rpc-baseline/spec.md` (6 added, 0 removed, 0 modified) + R17 登记; Iter 000 / Cycle 000 完成 T1 + T2 + Plan Review 接受的字段说明补 `amp` 同样在当前 Cycle 内部完成, 末轮 Plan Review accepted; Iter 001 / Cycle 000 → 001-rework → 002-rework 三轮闭环, 末轮 BUSY 全文扫描无锁/互斥保证或无条件 NOTIFY 相反表述, Plan Review accepted, `Next Cycle: None` / `Next Iteration: None`; 7/7 tasks 勾选, Persisted Evidence `none`, 未触动 `others/` 或既有 staged 内容。MS06 / MS07 已完成的产品交付 (2 篇 dma + 2 篇 network 文档 + 68 + 70 URL 覆盖 + G5 partial + delta spec + R15/R16 登记) 在本次同步基线之上保留, 工作区 `main` 上 staged 未 commit 的三 change 产物 (MS06 + MS07 + MS08) 仍待常规 commit。
- 任何字段变更需要刷新本文件并把状态从 `current` 保留, 或在新版本失效时标 `stale`。
