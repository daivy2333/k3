## 1. Iteration 000 — 板级事实层次与资源矩阵

- [x] 1.1 [T1] 在 `docs/reference/source-coverage.md` 登记 K3 datasheet 的官方 GitHub 对应页。
- [x] 1.2 [T2] 在 `docs/reference/source-coverage.md` 登记 CoM260 datasheet 的官网页与同页官方 GitHub 对应页。
- [x] 1.3 [T3] 在 `docs/reference/source-coverage.md` 登记 CoM260 user guide 的官网页与同页官方 GitHub 对应页。
- [x] 1.4 [T4] 在 `docs/reference/source-coverage.md` 登记 CoM260 hardware resources 的官方 GitHub 对应页，并同步实际 URL 行数说明。
- [x] 1.5 [T5] 创建 `docs/platform/k3-soc-overview.md`，形成只到 SoC 层的 CPU/hart、内存控制器、存储、外设与启动能力基线。
- [x] 1.6 [T6] 创建 `docs/platform/com260-board-resources.md`，形成 SoC、模组、Kit/载板和未知边界四层资源矩阵。

> 2026-09-07 第二轮修复: T4-T6 撤销勾选, 本轮 GREEN 后重新勾选. T1-T3 保持完成.

## 2. Iteration 001 — 启动链、镜像与 DTS

- [x] 2.1 [T7] 在 `docs/reference/source-coverage.md` 登记 K3 Buildroot `boot.md` 的官方 GitHub 对应页。
- [x] 2.2 [T8] 在 `docs/reference/source-coverage.md` 登记 K3 Buildroot `image.md` 的官方 GitHub 对应页。
- [x] 2.3 [T9] 在 `docs/reference/source-coverage.md` 登记 `linux-6.18` 的 K3 DTS 目录，并同步最终 URL 行数说明。
- [x] 2.4 [T10] 创建 `docs/boot/com260-boot-chain.md`，整理启动模式、介质优先级、Boot ROM、FSBL/SPL、ESOS、OpenSBI、U-Boot 与 payload/OS 的证据边界。
- [x] 2.5 [T11] 创建 `docs/boot/com260-image-and-dts.md`，整理镜像制品、写入方式、DTS 候选、目标映射边界与 CMA 两 cell 解码。
- [x] 2.6 [T12] 按实际 DTS 证据更新 G3 为 partial，新增 G7 CoM260 Kit 默认目标 DTS 未唯一映射；首轮尝试新增的 G8 超出 Task Contract 已删除。
- [x] 2.7 [T13] 更新 `docs/index.md`，链接四篇正文并把 platform 与 boot 状态改为已聚合。

## Task Contracts

### T1：K3 datasheet 交叉验证 URL 可精确追溯

- Requirement/Scenario: R1/S1，R2/S1-S2，R5/S1。
- Depends on: None.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 只有官网 K3 datasheet URL 和 `docs-chip` 仓库根 URL。
- Required behavior: 唯一登记 `https://github.com/spacemit-com/docs-chip/blob/main/zh/key_stone/k3/k3_docs/k3_ds.md`，身份为 `supporting-source`、范围为 `K3-common`、主题为 `platform`、状态为 `supporting`。
- Preserve: 官网行、其他 38 行及其观察日期；M01 权威边界。
- Forbidden: 不把 GitHub 页标为 `official-doc`，不刷新既有行。
- Test witness: 精确 URL 在修改前不存在；添加后恰好出现一次。
- GREEN condition: 行字段、实际观察日期、访问状态和备注均由当前证据支持。
- Verification: URL 精确匹配计数为 1；覆盖表 URL 无重复；`git diff --check` 通过。
- Stop when: 页面身份或 K3 路径不能确认。

### T2：CoM260 datasheet 两种来源身份分离

- Requirement/Scenario: R1/S2，R2/S1-S4，R5/S1。
- Depends on: T1.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: CoM260 datasheet 的精确 URL 未登记。
- Required behavior: 登记官网 `.../hardware/eco/k3_com260/com260_ds.md` 为 `official-doc`，登记 `https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_ds.md` 为 `supporting-source`；两行均为 CoM260/platform/current，聚合状态分别为 active/supporting。
- Preserve: 官网正文与 GitHub 交叉验证的证据等级差异。
- Forbidden: 不用 GitHub 文档内的 V1.2 自动改写不可读官网行的源端修订。
- Test witness: 两个精确 URL 修改前均不存在。
- GREEN condition: 两行各唯一存在，修订和访问状态与各自直接观察一致。
- Verification: 两个 URL 各计数 1；字段核对；URL 无重复。
- Stop when: 不能确认两者对应同一 CoM260 datasheet 身份。

### T3：CoM260 user guide 两种来源身份分离

- Requirement/Scenario: R1/S3，R2/S1-S4，R5/S1。
- Depends on: T2.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: user guide 的官网与 GitHub 精确 URL 均未登记。
- Required behavior: 登记官网 `.../hardware/eco/k3_com260/com260_user_guide.md` 为 active `official-doc`，登记 `https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_user_guide.md` 为 supporting-source；备注区分开发套件资料和交叉验证。
- Preserve: 产品版本、文档修订和观察日期三个概念分离。
- Forbidden: 不因 user guide 中出现 `v03` 就认定任何 DTS 匹配。
- Test witness: 两个精确 URL 修改前均不存在。
- GREEN condition: 两行唯一、身份正确，GitHub 页可证时记录文档内修订而非推给官网行。
- Verification: 精确 URL、字段和唯一性检查。
- Stop when: 页面并非 CoM260 Kit user guide 或目标身份无法确认。

### T4：hardware resources 交叉验证 URL 与覆盖计数一致

- Requirement/Scenario: R1/S2-S3，R2/S1-S4，R5/S1。
- Depends on: T3.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 只有 hardware-resources 官网 URL 和 `docs-product` 根 URL，表头仍声明固定 38 行。
- Required behavior: 登记 `https://github.com/spacemit-com/docs-product/blob/main/zh/k3_com260/com260_hw_resources.md` 为 CoM260/platform/supporting，并把表头行数说明改成 T1-T4 完成后的实际唯一 URL 数。
- Preserve: 原官网行和未选择行不变。
- Forbidden: 不把可下载设计文件内容视为已观察，除非实际打开并另行登记其 URL。
- Test witness: 精确 URL 不存在，且完成新增后“38 行”将与实际数量不一致。
- GREEN condition: 新行唯一，声明数等于表格数据行数，重复 URL 为 0。
- Verification: 提取表格 URL 后核对总数、唯一数和声明数；`git diff --check`。
- Stop when: 需要引用未登记的下载制品才能完成本 Iteration。

### T5：K3 SoC 能力与板级可用性解耦

- Requirement/Scenario: R1/S1、S4，R2/S3，R3/S1、S3，R5/S2-S3、S5。
- Depends on: T1-T4.
- Targets: `docs/platform/k3-soc-overview.md`。
- Current behavior: 文件和 `docs/platform/` 目录不存在；SoC 能力仅散见来源与分析。
- Required behavior: 首行引用已登记来源；按 CPU/hart、内存控制器、启动能力、存储/外设能力和后续主题边界整理 SoC 事实；每项标证据等级，并明确能力不证明 CoM260 引出、启用或软件可用 hart。
- Preserve: 简体中文、术语表主写法、500 行限制和 M01 权威边界。
- Forbidden: 不写 CoM260 连接结论，不展开寄存器、IRQ、DMA 或 GMAC 驱动细节，不引用非目标板。
- Test witness: 文件不存在；首行、必要章节和证据标记检查修改前失败。
- GREEN condition: 所有可证 SoC 事实有来源；板级可用性全部转交 T6 或标未知；文件少于 450 行，或按模板拆分。
- Verification: 检查首行、章节、四级证据用语、禁用板型词和相对链接；`git diff --check`。
- Stop when: 来源冲突会改变 K3 身份、CPU/hart 基本结论或需引入新事实类别。

### T6：CoM260 八类资源形成四层矩阵

- Requirement/Scenario: R1/S2-S4，R2/S1-S4，R3/S1、S3，R5/S2-S5。
- Depends on: T5.
- Targets: `docs/platform/com260-board-resources.md`。
- Current behavior: 文件不存在，SoC、模组和 Kit 资源没有统一归属表。
- Required behavior: 首行引用已登记来源；分别记录 CPU/hart、DRAM、UFS、SPI Flash、TF Card、debug UART、GMAC/PHY、连接器和供电的 SoC/模组/Kit 事实、证据等级与未知边界；版本冲突并列记录。
- Preserve: G3-G6 的既有缺口及后续 MS04-MS07 边界。
- Forbidden: 不把 SoC 能力写成板载资源，不把 Pico/其他变体写入 CoM260 结论，不推定 GMAC 实例或 PHY 参数。
- Test witness: 文件不存在；八类资源和三层标签检查修改前失败。
- GREEN condition: 八类资源无遗漏，每项可区分 SoC、模组、Kit/载板和未知边界；未知项含解除条件与影响主题。
- Verification: 检查首行、八类关键词、层级列、证据等级、G3 引用、禁用推断和行数。
- Stop when: 无法区分 CoM260 模组与 Kit，或来源冲突需要改变 Gate 1 范围。

### T7：boot 文档交叉验证 URL 可精确追溯

- Requirement/Scenario: R3/S1-S4，R5/S1。
- Depends on: Iteration 000 accepted.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 只有 boot 官网 URL 和 `docs-buildroot` 根 URL。
- Required behavior: 唯一登记 `https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/device/boot.md` 为 K3-common/boot/supporting。
- Preserve: 官网行及其状态。
- Forbidden: 不把仓库正文提升为官网事实。
- Test witness: 精确 URL 修改前不存在。
- GREEN condition: 新行字段和当前观察可证。
- Verification: URL 计数 1、字段和唯一性检查。
- Stop when: 页面路径或文档身份不能确认。

### T8：image 文档交叉验证 URL 可精确追溯

- Requirement/Scenario: R4/S1，R5/S1。
- Depends on: T7.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 只有 image 官网 URL 和仓库根 URL。
- Required behavior: 唯一登记 `https://github.com/spacemit-com/docs-buildroot/blob/main/zh/k3_buildroot/image.md` 为 K3-common/boot/supporting。
- Preserve: 官网事实边界和原行状态。
- Forbidden: 不把示例制品名当作稳定板型身份。
- Test witness: 精确 URL 修改前不存在。
- GREEN condition: 新行唯一且字段可证。
- Verification: URL、字段和唯一性检查。
- Stop when: 页面身份不能确认。

### T9：K3 DTS 候选集合有唯一来源入口

- Requirement/Scenario: R4/S2-S3，R5/S1。
- Depends on: T8.
- Targets: `docs/reference/source-coverage.md`。
- Current behavior: 只有 `linux-6.18` 仓库根 URL，未登记 K3 DTS 目录。
- Required behavior: 登记 `https://github.com/spacemit-com/linux-6.18/tree/k3-br-v1.0.y/arch/riscv/boot/dts/spacemit` 为 K3-common/platform+boot/supporting，并同步最终实际 URL 行数说明。
- Preserve: 分支名只作 SDK baseline，目录内容只作交叉验证。
- Forbidden: 不因文件名选择唯一目标 DTS；不登记或引用 Pico 文件为替代。
- Test witness: 精确目录 URL 修改前不存在。
- GREEN condition: 目录行唯一，总数说明与实际行数一致。
- Verification: URL、分支、字段、总数和唯一性检查。
- Stop when: K3 分支或 DTS 目录身份变化。

### T10：启动阶段和介质形成可追溯链

- Requirement/Scenario: R3/S1-S4，R1/S1-S3，R5/S2-S3、S5。
- Depends on: T7-T9.
- Targets: `docs/boot/com260-boot-chain.md`。
- Current behavior: 文件和 `docs/boot/` 目录不存在。
- Required behavior: 区分 SoC 支持与板级可用，记录 download/local boot、已证介质与优先级，以及 Boot ROM → OpenSBI → U-Boot → payload/OS 的可证顺序、输入输出和版本边界。
- Preserve: MS02 SDK baseline；未知项闭包；官方事实与交叉验证分离。
- Forbidden: 不推定 load address、DRAM reserved 区或 handoff 寄存器，不把所有 SoC 介质写成 Kit 可用。
- Test witness: 文件不存在；阶段、介质、来源和未知项检查修改前失败。
- GREEN condition: 阶段与介质均可追溯，版本不一致处显式标注，缺失参数保留未知。
- Verification: 首行、阶段、介质、证据等级、禁用常量和行数检查。
- Stop when: 来源给出的阶段顺序实质冲突且无法按版本分层。

### T11：镜像、写入方式与 DTS 非唯一性明确

- Requirement/Scenario: R4/S1-S3，R3/S3-S4，R5/S2-S3、S5。
- Depends on: T10.
- Targets: `docs/boot/com260-image-and-dts.md`。
- Current behavior: 文件不存在，镜像与目标 DTS 没有稳定说明。
- Required behavior: 记录可证镜像类型和写入方式；列出 CoM260 DTS 候选集合；只有直接打开相应文件后才写 compatible/include/chosen/memory/aliases；不能唯一映射时给出缺失影响和解除条件。
- Preserve: 目标板为 CoM260 Kit；DTS 仅属交叉验证。
- Forbidden: 不选 Pico/通用 K3 DTS，不把文件名或 user-guide 产品版本当映射证明，不推定地址。
- Test witness: 文件不存在；镜像、DTS 候选、非唯一边界和证据标记检查修改前失败。
- GREEN condition: 镜像流程和 DTS 边界完整；唯一/非唯一两个分支均按证据执行。
- Verification: 首行、制品、写入、候选、解除条件、禁用板型和行数检查。
- Stop when: 来源集合出现未规划的新目标板身份，或唯一映射需要修改批准范围。

### T12：DTS/GMAC 缺口不重复且与证据一致

- Requirement/Scenario: R2/S2-S4，R4/S2-S3，R5/S4-S5。
- Depends on: T11.
- Targets: `docs/reference/known-gaps.md`。
- Current behavior: G3 覆盖 GMAC/PHY 参数，但 G1-G6 不覆盖目标 Kit 与唯一 DTS 的映射。
- Required behavior: 若精确 DTS 证据改善 G3，则只补充 G3 当前证据并保持未解除字段；若目标 Kit 映射仍不唯一，新增完整且不重复的 G 条目和汇总；若两者均无适用变化，记录 `SKIPPED: no new or changed gap`。
- Preserve: G1-G6 历史、编号连续、状态枚举和缺口四要素。
- Forbidden: 不因文件名宣告缺口关闭，不创建 G3 同义项，不删除历史。
- Test witness: 非唯一映射成立时，相应独立 G 条目修改前不存在；不触发时文件 diff 必须为空。
- GREEN condition: 缺口表与 T11 结论完全一致，无重复或虚假关闭。
- Verification: 对照 T11 检查 G3、新 G、汇总和交叉引用；`git diff --check`。
- Stop when: 证据要求关闭 G3 但不能满足其全部解除条件。

### T13：四篇主题正文从总入口可达

- Requirement/Scenario: R5/S2-S3、S5。
- Depends on: T5、T6、T10-T12.
- Targets: `docs/index.md`。
- Current behavior: platform/boot 仅列为未来职责，没有主题正文链接。
- Required behavior: 新增四个相对链接；把 platform 和 boot 状态改成与实际交付一致；保持其他七类主题仍为未来路径。
- Preserve: 总入口的 CoM260 范围、reference 链接和未交付主题状态。
- Forbidden: 不创建空 overview，不把未来主题标为已聚合。
- Test witness: 四个相对链接修改前均不存在。
- GREEN condition: 四个链接均解析到真实文件，职责描述与正文一致。
- Verification: 解析相对链接；检查四个目标存在、其他主题状态未误改；完整 diff Review。
- Stop when: 任一主题文档未达到 GREEN 或发生拆分而导航目标未重新规划。
