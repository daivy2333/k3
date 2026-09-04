> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs（源端修订: unknown；观察日期: 2026-09-02）

# 已知缺口

> 适用范围: 登记 `docs/` 当前无法支撑的官方资料或硬件事实，每项含解除条件。
> 产品文档通过获批 OpenSpec change 修改；change accepted 后，docs-maintainer 仅按实际结果同步 SNAPSHOT、tasks、M/D/K/R/I 中适用的状态。
> 引用来源: 当前六类缺口与 `source-coverage.md` 的 `unknown` / `unverified` / `deferred` 状态保持一致。

## 缺口登记规则

- 每类缺口必须包含：当前证据、禁止推断、解除条件、影响主题。
- 缺口解除后必须保留历史记录，记录解除日期与依据来源；不静默删除。
- 新增缺口需在主题文档写作前先登记到本表，不得在主题文档中临时声明资料不足。
- 缺口状态：`open`（待解除）/ `partial`（部分解除，仍有未解事实）/ `closed`（已解除）。

---

## G1. 官网主题页总数未知

- 分类: 范围盘点
- 当前证据: R03 分析（2026-09-02 捕获）记录 K3 主路径至少 52 篇非索引主题页；R04-R08 覆盖其中 37 个唯一 URL。
- 禁止推断: 不得用 R04-R08 的 37 个 URL 直接声称「K3 共有 37 篇或 52 篇」；不得假定未观察子树已完整。
- 解除条件:
  - 完整遍历 R01 入口的全部子树并记录页面标题与 URL；
  - 与 R08 中 `docs-chip` / `docs-product` / `docs-buildroot` 仓库目录交叉对比；
  - 形成「总页数 = N」的事实描述，并写入 `source-coverage.md` 的备注或独立 `references/spec.md` 条目。
- 影响主题: 全部 `docs/` 主题文档；任何「全 K3 资料盘点」类主题均依赖本缺口解除。

## G2. 官网未完整展开目录

- 分类: 范围盘点
- 当前证据: R03 记录 USB、RV2768、Shelf、dpdk、esos、kernel_debug 和 media 等目录未完整展开；其它可能存在的子树同样未知。
- 禁止推断: 不得以当前覆盖表为完整集合推论 K3 不存在其它资料；不得在主题文档中暗示「其余内容无需关心」。
- 解除条件:
  - 对 R01 入口逐级展开并记录每个目录子树；
  - 与 R08 的目录结构对照，确认未覆盖子树；
  - 对暂不聚合内容（如 RV2768、Shelf）保持 `out-of-scope`，对未观察内容继续登记为缺口。
- 影响主题: 全部 `docs/` 主题文档；尤其是 platform 与 buses 主题，需确认是否存在未观察的 PCIE 子页面。

## G3. K3 CoM260 实际 GMAC 实例与 PHY 详情

- 分类: 硬件事实
- 当前证据: R05 的 09-GMAC.md 仅说明 K3 SDK 中 GMAC 驱动的通用用法；R04 的 com260_hw_resources.md 提供 CoM260 模组硬件资源列表，但未直接确认 GMAC 实例编号、MMIO 基地址、PHY 型号、MDIO 地址、PHY 地址、RGMII delay、reset 与 ref clock。
- 禁止推断: 不得基于 Linux 驱动行为或 DWMAC 默认值反推 CoM260 实际硬件；不得假定 GMAC 实例与 Pico 板相同。
- 解除条件:
  - 取得 CoM260 模组 hardware resources 中关于 GMAC/PHY 的具体字段（实例号、MMIO、PHY 型号、MDIO/RGMII 参数）；
  - 或取得 CoM260 DTS（device tree）源文件并对照 compatible、reg、phy-handle、phy-mode；
  - 形成 R04 子条目的修订记录或新增 R 登记。
- 影响主题: `docs/network/`（GMAC、PHY、MDIO、RGMII）；间接影响 `docs/platform/`（clock、reset、pinctrl）。

## G4. AIA / APLIC / IMSIC 地址、IRQ domain 与 hart delivery

- 分类: 硬件事实
- 当前证据: R05 的 16-Clock.md、Reset.md、Timer.md 涉及 K3 clock/reset/timer 但未公开 APLIC 与 IMSIC 的 MMIO 地址、IRQ domain 拓扑、hart routing 规则、mask/ack/complete 协议细节。
- 禁止推断: 不得用 RISC-V AIA 标准行为直接等同 K3 实现；不得假定 K3 APLIC/IMSIC 寄存器布局与 SiFive 或其他厂商一致。
- 解除条件:
  - 取得 K3 SoC 公开寄存器手册（programmer reference 或 datasheet 寄存器章节）；
  - 或在 R08 的 linux-6.18 仓库定位 K3 APLIC/IMSIC 设备树与驱动初始化源码；
  - 形成 address、IRQ domain、hart/context、delivery 等事实条目。
- 影响主题: `docs/interrupts/`（AIA、APLIC、IMSIC、timer）；间接影响 `docs/serial/` 与 `docs/network/`（驱动 IRQ 接入）。

## G5. DMA coherency、IOMMU、cache line 与 barrier 规则

- 分类: 硬件事实
- 当前证据: R05 的 21-DMA.md 仅说明 K3 SDK 中 DMA 驱动的通用配置；未公开 K3 SoC 的 cache 一致性模型、IOMMU 是否存在、DMA 地址宽度、descriptor 与数据缓冲区的 ownership 转换规则、cache line 大小、barrier 要求。
- 禁止推断: 不得用通用 RISC-V 行为或 Linux DMA API 行为等同 K3 实际；不得假定 K3 默认全 coherent 或全 non-coherent。
- 解除条件:
  - 取得 K3 SoC 公开寄存器手册中 cache、MMU、IOMMU 章节；
  - 或在 R08 的 linux-6.18 仓库定位 K3 IOMMU 与 DMA 一致性相关 patch；
  - 形成 coherency model、IOMMU presence、barrier list 与 ownership 转换条目。
- 影响主题: `docs/dma/`；间接影响 `docs/network/`（GMAC descriptor、buffer）、`docs/storage/`（UFS/SDHC）。

## G6. K3 GMAC 寄存器、descriptor 与 interrupt ack 的 programmer reference

- 分类: 硬件事实
- 当前证据: R04 的 k3_ds.md、root_overview.md 是产品层与 overview 层资料；R05 的 09-GMAC.md 是驱动使用说明；均不替代 GMAC 寄存器、descriptor ring、interrupt cause/mask/ack 寄存器级描述。
- 禁止推断: 不得以 Linux DWMAC 行为或 Synopsys DWMAC 通用寄存器定义反推 K3 GMAC；不得假定 ack/cause 寄存器布局等同于上游驱动。
- 解除条件:
  - 取得 SpacemiT 官方 GMAC 寄存器手册（programmer reference）；
  - 或在 R08 的 linux-6.18 仓库定位 K3 GMAC 设备树与驱动的寄存器偏移表；
  - 形成寄存器偏移、复位值、interrupt cause/mask/ack 与 descriptor 字段定义条目。
- 影响主题: `docs/network/`（GMAC 寄存器与 descriptor、interrupt ack）；间接影响 `docs/dma/`（descriptor buffer ownership）。

---

## 缺口状态汇总

| 编号 | 分类 | 状态 | 最近核对日期 |
| --- | --- | --- | --- |
| G1 | 范围盘点 | open | 2026-09-02 |
| G2 | 范围盘点 | open | 2026-09-02 |
| G3 | 硬件事实 | open | 2026-09-02 |
| G4 | 硬件事实 | open | 2026-09-02 |
| G5 | 硬件事实 | open | 2026-09-02 |
| G6 | 硬件事实 | open | 2026-09-02 |

## 缺口与 source-coverage 的对应

- G1、G2 对应 `source-coverage.md` 中观察日期 2026-09-02 的 38 行；未观察子树以「未登记」处理。
- G3、G4、G5、G6 对应 `source-coverage.md` 中标记为 `active` 但未提供寄存器级描述的 R04/R05 资料行。
- R06 的 15 个 `deferred` URL 暂不展开到本表；如后续进入聚合 change，再决定是否新增对应 G 条目。
