## Context

仓库目前没有 `docs/storage/` 正文。[总索引](../../../docs/index.md)把 QSPI、SPI、SDHC 和 UFS 标为 `deferred`；[来源覆盖表](../../../docs/reference/source-coverage.md)已有四个官网入口，其中 UFS 另有已观察的 GitHub supporting source。MS03 已记录启动介质和 CoM260 板级资源，MS06 已记录通用 DMA/cache/ownership 边界。本 change 需要增加存储专属解释，但不能复制或改写这些既有职责。

调查确认两类软件材料：固定 revision `19219411d5dc1515496f910d04c93da12ee95be4` 的第三方 tgoskits 含 K3 UFS 同步 block driver，也含未接入该板 profile 的 portable `k3-sdhci` core。前者可解释 MPHY/UniPro、UTP/SCSI、轮询完成和 controller recovery；后者可解释 K3 vendor SDHCI PHY、HS200/HS400 和 tuning，但不能证明 CoM260 当前运行该实现。普通 SPI/QSPI 没有同等级 K3 第三方数据面实现，必须以官方资料和 DTS 静态事实为边界。

## Goals and Non-goals

### Goals

- 为 QSPI、SPI、SD/eMMC 和 UFS 提供按设备分层的资源、启动、数据路径和故障边界。
- 让静态能力、板级连接、官方软件行为、第三方实现和本项目未验证行为可区分。
- 把 UFS 的 descriptor、DMA/cache、轮询、超时和恢复路径说明到可用于后续实现调查的程度。
- 维持来源覆盖、术语、缺口和索引的一致性。

### Non-goals

- 不实现驱动，不运行真板、刷写或介质写测试。
- 不把 `k3-sdhci` portable core 表述为当前 CoM260 board profile 已接入的运行路径。
- 不把 UFS 的协议、DMA、轮询或恢复语义外推到 QSPI、SPI、SDHC。
- 不选择 G7 尚未唯一映射的 CoM260 顶层 DTS 作为默认目标。

## Decisions

### D1：两篇正文按故障域拆分

创建 `docs/storage/k3-qspi-spi-sdhci.md` 和 `docs/storage/k3-ufs.md`。前者覆盖 QSPI、普通 SPI、SD/eMMC/SDHCI，并在设备内部继续分层；后者独立覆盖 UFS。替代方案是按四个控制器各建一篇文档，但普通 SPI/QSPI 当前证据不足以形成两个稳定、独立的长文，易产生空壳。另一替代方案是全部合并，UFS 的协议栈和恢复细节会使其他设备边界难以检索。

### D2：先建立来源责任和不可达边界，再写正文

Iteration 000 先把四个官网入口从 `future/deferred` 调整为当前存储职责。官网 SPA 与三个尚未登记的 docs-buildroot raw 页面在当前环境不可直接读取，因此保留 `partially-observed` 或明确不可达状态；只有实际打开的 URL 才能新增 supporting row。正文以可读的官方 docs-chip K3 datasheet、既有已观察 UFS supporting row、既有主题文档及固定第三方源码为依据，缺少的 driver 细节进入未知项。这样不会把网络能力问题伪装成硬件结论，也不把取证留给 Act 临场决定。

### D3：启动关系只做交叉引用和差异补充

存储正文引用 `com260-boot-chain.md` 与 `com260-image-and-dts.md`，只补充控制器视角的介质能力、阶段和数据路径，不复制完整启动链。若新证据与既有启动顺序冲突，Act 停止并返回 Plan，不静默覆盖 MS03。

### D4：静态资源不等于运行时路径

DTS 的 `compatible`、MMIO、IRQ、clock/reset、DMA 属性只证明该变体的静态配置。第三方 UFS 驱动解析 IRQ 但不注册 handler，UIC 和 transfer completion 均轮询；文档必须保留该差异。`k3-sdhci` 只提供 portable vendor core，OS glue、FDT probe、IRQ 和 block registration 由消费层负责，因此不能据此声明当前 board profile 已启用。

### D5：错误恢复按可观察状态描述

UFS 文档区分 init/link failure、UIC timeout、transfer timeout、OCS error、fatal interrupt 和 recovery failure。固定实现会在 timeout/OCS/fatal 后做 HCE reset、重建 link/list、NOP 验证并重试一次；recovery 失败后 latch fatal，后续请求快速失败。文档不把此第三方策略升级为 K3 硬件保证，也不声称未完成写入可安全重试。

### D6：验证只检查知识产物行为

测试见证是目标文件缺失、来源仍 deferred、导航未接入等当前 RED。GREEN 通过首行来源、必需章节、证据等级、未知项四字段、相对链接、来源覆盖唯一性、术语/计数一致性和 `openspec validate --strict` 直接观察。仓库无产品代码，不新增验证脚本或 Evidence 目录。

## Current-State Evidence

| Surface | Current state | Evidence |
|---|---|---|
| `docs/storage/` | 不存在；索引状态为待聚合 | `docs/index.md:64`，文件树检查 |
| 官网入口 | QSPI、SDHC、SPI、UFS 四行均为 `future / deferred / partially-observed`；当前环境访问 raw 对应页因 DNS 失败，web fetch 为 cache miss | `docs/reference/source-coverage.md:49-50,58-59`；2026-09-10 `curl` exit 6 |
| UFS supporting source | 已登记 docs-buildroot raw UFS 页并标记 observed | `docs/reference/source-coverage.md:81` |
| SoC 能力 | QSPI 支持 XIP/Page、1/2/4 线和 NOR/NAND；SD/eMMC 为 SDHCI compatible；UFS 2.2/UniPro 1.6/M-PHY 3.0 | `docs/platform/k3-soc-overview.md:67-74`；官方 docs-chip K3 datasheet |
| 板级资源 | CoM260 记录板载 UFS、SPI Flash、TF 卡及 UFS 128/256 GB 冲突 | `docs/platform/com260-board-resources.md:53-83,195-200,262-268` |
| 启动关系 | SD 优先及 eMMC/SPI NOR/SPI NAND/UFS 候选、固件布局已在 MS03 建立 | `docs/boot/com260-boot-chain.md:39,64-109`；官方 docs-chip K3 datasheet |
| 固定 DTS | IFX DTS 含 3 个 `spacemit,k3-sdhci` 节点、1 个 `spacemit,k3-qspi`、多个普通 SPI 和 `spacemit,k3-ufshcd`；只证明该第三方变体 | `others/Rt-Async-AMP/tgoskits/os/StarryOS/configs/board/spacemit-k3-com260-ifx.dts` |
| SDHCI 软件材料 | `k3-sdhci` core 负责 K3 PHY、SD/eMMC mode、HS200/HS400、DLL 和 software RX tuning；不负责 OS glue | `others/Rt-Async-AMP/tgoskits/drivers/blk/k3-sdhci/src/lib.rs`、`vendor_ext.rs` |
| UFS 软件材料 | probe → host/MPHY/UniPro/link → transfer lists → device init → LUN scan → sync block registration | `others/Rt-Async-AMP/tgoskits/drivers/ax-driver/src/block/k3_ufs/mod.rs` |
| UFS completion/recovery | IRQ 只解析不注册；UIC/transfer 轮询；timeout/OCS/fatal 后 recovery + 单次重试，失败 latch fatal | `k3_ufs/transfer.rs:122-126,255-303,419-596`、`uic.rs`、`error.rs` |
| 既有缺口 | G5 涉及 storage DMA/cache，G7 涉及默认 Kit DTS；尚无存储专属 programmer/runtime 缺口 | `docs/reference/known-gaps.md` |

## Data and Responsibility Flow

```text
official source / official GitHub supporting source
                    │
                    ▼
          source-coverage responsibility
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
 QSPI/SPI/SDHCI topic      UFS topic
          │                   │
          └─────────┬─────────┘
                    ▼
 index + terminology + known gaps
```

文档数据流不改变运行状态。Act 只读取外部材料并写 Markdown；来源不可达时记录访问状态和 supporting evidence，不生成抓取缓存。

## Risks

- 官网 SPA 页面仍只能部分观察，三个新 raw 对应页当前也不可达；正文必须保留 M01 权威入口，只把已实际打开的 GitHub 页面标为 supporting，并把缺少的 driver 细节列为未知项。
- 官方 docs-chip 英文页可能比 SNAPSHOT 的 2026-09-08 观察基线更新；若内容变化影响既有事实，应返回 Plan 判断 refresh，而不是在本 change 静默提升全局源端日期。
- 固定第三方 DTS 名称为 IFX，受 G7 约束；其地址和 IRQ 不能代表默认 Kit。
- QSPI 与普通 SPI 的驱动模型不同；前者面向 SPI memory/XIP，后者是通用控制器。正文必须避免把 QSPI 当作任意 SPI message controller。
- eMMC 与 SD 共用 SDHCI 家族但板级介质、bus width、PHY/timing 和 removable 属性不同。

## Verification Strategy

- 每个任务先以目标文件缺失、行状态 deferred 或入口缺失形成 RED/变更前 GREEN 见证。
- 对新增正文检查首行来源、设备分层、证据等级、错误边界、未知项字段和相对链接。
- 对覆盖表检查 URL 唯一、四个官网入口状态、supporting rows 和主题职责。
- 对收尾文件检查索引可达、术语唯一、G 条目与汇总计数一致。
- 每个 Iteration 运行相对链接检查、`git diff --check` 和针对该 Iteration 的内容断言；change 级运行 `openspec validate establish-k3-storage-controller-baseline --strict`。
- 全量 `openspec validate --all --strict` 当前因两个既有 spec 失败，只作回归对照；本 change 不修复无关 spec。
