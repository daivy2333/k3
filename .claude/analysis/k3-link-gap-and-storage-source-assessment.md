# K3 链接缺口与存储来源评估

> Snapshot: [SNAPSHOT](../docs/SNAPSHOT.md)
> Captured revision: `820535c0bab7b2e58df3c1c01bc6a9e2689ba4a9`
> Observed branch: `main`
> Captured at: 2026-09-11
> See also: [K3 官方资料聚合分析](k3-official-docs-for-starryos-async-drivers.md)、[StarryOS 复用清单](rt-async-amp-k3-starryos-reuse.md)

## 结论

临时清单中的 24 个规范化 URL 有 4 个已在 `source-coverage.md` 逐 URL 登记，4 个已有官网入口或仓库级记录覆盖，16 个尚未逐 URL 登记。后 16 个主要服务于镜像制作、烧录、OpenSBI、StarryOS/ArceOS 移植和工具链调查，不是当前 `establish-k3-storage-controller-baseline` Iteration 000 的来源缺口。

当前 Iteration 真正需要补查的是 QSPI、SPI、SDHCI 三个 docs-buildroot 对应页，以及 Linux K3 分支中的具体 DTS、binding 和 driver 文件。临时清单只提供了 docs-buildroot 与 linux-6.18 仓库入口，没有定位这些直接证据。

本次对 GitHub raw URL 的访问均因 `Could not resolve host: raw.githubusercontent.com` 失败，退出码为 6。因此，未登记候选只能保留为待核对入口，不能标记为当前已观察，也不能据其摘要补写产品事实。

## 目标与范围

本分析回答四个问题：

1. 临时清单中的链接是否已被当前来源覆盖表吸收。
2. 未登记链接是否属于当前 MS09 / Iteration 000。
3. 哪些候选适合未来 StarryOS 真机移植或镜像工作。
4. 当前存储 change 还缺哪些直接来源。

范围仅包含链接去重、用途分类、当前 change 适用性和可达性。不验证网页摘要中的硬件结论，不修改 `docs/reference/source-coverage.md`，也不扩大已批准 Cycle 的任务范围。

## 候选链接与覆盖状态

查询参数 `utm_source=chatgpt.com` 已从比较键中移除。仓库首页与仓库内具体文件按不同 URL 处理。

### 已逐 URL 登记

| 候选 URL | 现有职责 | 当前用途 |
|---|---|---|
| `docs-product/.../com260_user_guide.md` | CoM260 platform | 可复用板载 SPI Flash、TF Card、UFS 和接口可达性；不是新来源 |
| `docs-product/.../com260_ds.md` | CoM260 platform | 可复用模组存储能力和型号边界；不是新来源 |
| `docs-buildroot/.../image.md` | K3 boot | 只支持镜像与刷机关系；当前存储正文可交叉引用，不重复定义 |
| `spacemit-com/linux-6.18` | K3 workflow-support | 仅为仓库入口；具体存储事实仍需文件级 URL |

### 已有语义覆盖，但 URL 不完全相同

| 候选 URL | 已有覆盖 | 处理建议 |
|---|---|---|
| `docs-buildroot/.../source.md` | 官网 `source.md` 入口与 docs-buildroot 仓库级 R08 | 保留为 SDK 构建入口；除非直接打开 GitHub 页面，否则不新增 observed row |
| `docs-buildroot/tree/main/zh/k3_buildroot` | docs-buildroot 仓库级记录 | 目录页不替代 QSPI/SPI/SDHCI 具体页面 |
| `docs-chip/.../en/.../k3_ds.md` | 已登记中文 `zh/.../k3_ds.md` | 英文页是独立 URL；只有直接读取并确认版本、差异和用途后才登记 |
| `spacemit-com` 组织页 | R08 的官方仓库集合 | 适合发现仓库，不作为设备事实来源 |

### 未逐 URL 登记，且不属于当前 Iteration 000

| 分组 | 候选 | 合理用途 |
|---|---|---|
| 镜像与烧录 | Titan Flasher、K3-Ubuntu-Images 仓库及 README | 未来 StarryOS 镜像封装、分区、fastboot/Titan 工作流调查 |
| 厂商启动组件 | `uboot-2022.10`、SpacemiT OpenSBI | 未来 K3 启动交接、S-mode payload 和固件兼容性调查 |
| OpenSBI 上游 | generic platform、platform guide、platform requirements | 解释 FDT、timer、IPI、PMP 和平台责任；不是 K3 专属证据 |
| OS 实现 | StarryOS、ArceOS、ArceOS app-helloworld | 未来移植的实际代码调查对象 |
| SDK 组成 | manifests、buildroot、buildroot-ext | 固定 SDK 仓库、分支、板级配置和补丁关系 |
| 其他发行版 | archlinux-spacemit | 对照第三方 OS 镜像流程，不能替代 K3 官方硬件材料 |
| 工具链 | `.github/upstream-status/toolchain.md` | 未来核对 X100、ISA 和编译器支持状态 |

这些链接适合作为未来分析的检索入口。只有在对应 change 中直接读取、记录版本或分支并形成具体用途后，才应进入 `source-coverage.md` 或 references。

## 当前 MS09 的链接缺口

### Iteration 000：应优先补查

覆盖表已有四个官网 SPA 入口，其中当前 Iteration 直接涉及：

- `07-QSPI.md`
- `08-SDHC.md`
- `SPI.md`

还需要从 docs-buildroot GitHub 对应目录定位这三个文件的 raw/blob URL，并从 linux-6.18 的 `k3-br-v1.0.y` 分支定位：

- K3 QSPI、普通 SPI、SDHCI 控制器节点及 CoM260 板级启用节点；
- K3 或所用控制器的 Device Tree binding；
- QSPI/SPI/SDHCI platform、host、PHY 或 tuning driver；
- 能区分静态能力、portable core 和实际板级启用状态的调用入口。

只有实际打开的具体 URL 才能新增 supporting row。仓库首页、搜索结果或临时清单摘要不能满足 T1 的直接观察条件。

### Iteration 001：已有入口，仍需按计划调查

`ufs.md` 的 GitHub raw URL已登记并在 2026-09-09 观察。后续 UFS Iteration 仍需核对具体 DTS、binding、driver 与固定第三方实现，但不属于当前 Cycle 000。

## 已确认事实、推断与未确认项

### 已确认事实

- `source-coverage.md` 当前声明 70 个唯一 URL。
- QSPI、SDHC、SPI、UFS 官网入口仍为 `future / deferred / partially-observed`。
- UFS GitHub raw 页面已经登记；计划明确指出其他三个 docs-buildroot raw 页面尚未登记。
- 当前 change 有 6 个任务；Iteration 000 只执行 T1、T2，UFS 和最终导航收敛被延后。
- 临时清单中的多数链接面向构建、刷写或 OS 移植，超出当前 Cycle 的产品范围。

### 推断

- K3-Ubuntu-Images、Titan、U-Boot、OpenSBI、StarryOS 和 ArceOS 可以组成未来真机移植的来源链，但需要独立 change 重新调查版本、入口和验证边界。
- manifests 与 buildroot-ext 可能比 Buildroot 主仓更适合定位厂商分支和板级配置；在未读取前不能确认其当前 K3 内容。

### 未确认项

- 16 个未登记 URL 在 2026-09-11 的实际内容、分支和最后修订状态。
- docs-buildroot 中 QSPI、SPI、SDHCI 对应 GitHub 文件的当前存在性和内容。
- linux-6.18 K3 分支中应登记哪些存储 binding/driver 文件。
- 英文 K3 datasheet 与已登记中文页是否同版本、同内容。
- 临时清单声称的 RVA23、IOMMU、EDK2 启动链等内容是否适用于当前 CoM260 基线；这些声明不得在未核对原文和版本时写入产品文档。

## 来源流与边界

```text
仓库/目录入口
    ↓ 定位具体页面或源码文件
直接读取并确认内容、分支、适用设备
    ↓
source-coverage 唯一 URL + 证据等级
    ↓
QSPI / SPI / SDHCI 正文事实或未知项
```

任一环节出现 DNS 失败、SPA 壳、版本不明或板级映射不唯一时，结果只能进入不可达说明或未知项，不能提升为已观察硬件事实。通用 OpenSBI、ArceOS 和发行版实现不能替代 K3 官方来源。

## 测试、验证入口与影响面

本次实际执行：

- 从临时文档提取并规范化 URL，得到 24 个唯一候选。
- 使用 `rg` 对照 `source-coverage.md`、references 和当前 change。
- 对 `source.md`、`image.md`、K3-Ubuntu-Images README 和 `k3.dtsi` raw URL执行 `curl --fail --max-time 20`；四次均因 DNS 解析失败退出 6。
- 检查当前 change tasks 与 Cycle，确认当前授权范围为 T1、T2，Persisted Evidence 为 `none`。

本分析可以作为当前 Act 的来源筛选输入，但不替代 Task Contract，也不授权新增未直接读取的 URL。未来 StarryOS 移植计划可以复用未登记候选分组，届时必须重新检查网页和仓库的新鲜状态。

## 关键文件

- [`docs/reference/source-coverage.md`](../../docs/reference/source-coverage.md)：当前 URL 覆盖与观察状态。
- [`openspec/changes/establish-k3-storage-controller-baseline/tasks.md`](../../openspec/changes/establish-k3-storage-controller-baseline/tasks.md)：MS09 任务和 Iteration Map。
- [`000-initial.md`](../../openspec/changes/establish-k3-storage-controller-baseline/iterations/000-storage-sources-and-sdhci/000-initial.md)：当前 Cycle 的来源规则、任务契约和停止条件。
- [`openspec/specs/references/spec.md`](../../openspec/specs/references/spec.md)：R04、R05、R08 及既有分析索引。
