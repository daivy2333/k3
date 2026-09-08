## Context

MS01 已建立 38 行唯一 URL 覆盖表和人工刷新指南。当前 7 个批准目标均存在于 `docs/reference/source-coverage.md`：R01 权威入口位于第 28 行，R07 两个 SDK 页面位于第 60–61 行，R08 四个官方 GitHub 仓库位于第 62–65 行。它们的观察日期都是 `2026-09-02`；R01/R07 为 `partially-observed`，R08 为 `observed`。

覆盖表拥有长期聚合状态、访问状态和观察元数据；一次 refresh 的逐 URL 结果只由当前 Cycle 的 Act Response 保存。初始执行证明：如果要求本轮新观察字段也必须有 2026-09-02 上次值，首次 refresh 将无法完成。返工设计因此区分“已持久化字段比较”和“新字段建立 baseline”，并继续保留五种互斥结果、两阶段修改边界和中断恢复。

2026-09-05 的 Plan 调查得到以下访问基线：仓库环境中的 `curl` 无法解析 spacemit.com 或 github.com；浏览来源可以定位四个 R08 公共仓库，SpacemiT 的 R01 与 `source.md` 返回 cache miss，release-notes 页面可定位但没有可读取正文。这些结果只证明当前工具路径的可用性，不是 Act 的正式刷新结论。Act 必须重新观察来源。

本 change 没有程序入口、调用链、并发状态或资源生命周期。关键路径是：读取覆盖行 → 取得当前来源信息 → 按批准职责比较 → 分类 → 更新单行元数据 → 在 Act Response 保存逐 URL 结果 → 验证未检查行和权威边界。

## Goals / Non-Goals

**Goals:**

- 对批准的 7 个 URL 完成一次可审计的真实 refresh。
- 使首次 refresh 在没有正文快照时只比较已持久化字段，并为新观察字段建立当前 baseline。
- 建立带证据等级的 SDK baseline，并保持 R08 只作 supporting source。
- 允许在任意 URL 后中断并从 `next` 继续。
- 以一个逻辑 Iteration 完成 MS02 change 的产品修改和验证。

**Non-Goals:**

- 不把整个页面或仓库的任意内容变化都纳入比较。
- 不聚合硬件、启动或驱动技术正文。
- 不刷新未批准的 31 个 URL。
- 不直接维护 SNAPSHOT、tasks、references、M/D/K/I 或 milestone 状态。
- 不创建抓取、diff、链接检查或证据身份工具。

## Decisions

### D1：首次 refresh 只比较已持久化字段

2026-09-02 已持久化的 URL、来源身份和访问状态是首次 refresh 的比较投影。导航可达性、K3 路径、默认或 K3 相关分支、SDK 版本和 release 正文没有上次值；本轮观察到这些字段时，将其记录为当前 baseline，但不据此判定 `unchanged` 或 `changed`。

R01 或 R07 即使 HTTP 成功，只要响应只有 SPA 壳且无法取得所需正文，就归类为 `unreachable`。只有在当前观察连已持久化的 URL 和来源身份也无法确认，且又不符合 `unreachable`、`moved` 或 `removed` 时，Act 才停止并返回 Plan。

### D2：来源访问遵循权威顺序

Act 先访问原 URL，再用 R01 入口或明确重定向判断 moved/removed，最后使用 R08 仓库历史交叉验证。R08 不得覆盖 R01/R07 的官网事实；差异写入逐 URL 结果和待解除条件。

替代方案是以 GitHub 可访问性替代官网访问。该方案违反 M01 和 R08 supporting 边界，因此不采用。

### D3：逐 URL 结果与长期状态分开保存

Act Response 使用固定顺序的结果表，至少记录 URL、当前观察、比较依据、五种结果之一、覆盖表变化、影响主题和 `next`。`source-coverage.md` 只保存实际允许的长期字段变化：

- `unchanged`：只更新观察日期；
- `changed`：更新观察日期、明确可证的源端修订和必要备注；
- `moved`：保留旧行与聚合/访问状态，在备注写迁移并新增完整 URL 行；
- `removed`：保留旧行与聚合/访问状态，在备注写下架；
- `unreachable`：更新观察日期，访问状态仅在新证据推翻旧证据时改变。

未检查行保持不变。一次刷新结果不写入覆盖表，也不增加全局“本轮完成”字段。

### D4：修订指南后重新执行真实 refresh

`source-refresh.md` 需要把初始执行写入的过严规则改为 D1 的持久字段比较、新字段建 baseline 和 SPA 壳分类规则。随后重新观察全部 7 个 URL。现有 2026-09-05 日期只有在新观察产生有效结果时才能保留；否则恢复为 2026-09-02。若出现有官方证据的 `removed`，或既有规则认可的长期不可访问且无替代来源，才修改 `known-gaps.md`；否则该条件任务明确跳过。

替代方案是只在 Cycle 中保存比较规则。该规则会影响以后所有首次或无快照 refresh，应进入产品指南，不能只存在于单次执行上下文。

### D5：一个 Iteration 承载 MS02

指南补充、7 URL refresh 和条件缺口登记共同形成一个来源追踪 baseline，依赖顺序明确且共享同一验证边界。拆成独立 Iteration 会让第一阶段只产生规则而没有真实刷新结果，不能形成 MS02 的稳定成果；因此本 change 只有 `Iteration 000: 首次来源刷新`。

### D6：验证结果保存在 Act Response

刷新结果、命令、每项不超过 20 行的决定性输出和退出码足以支持 Review。Persisted Evidence 使用 `none`。不保存网页副本、完整日志、commit Hash、manifest 或运行 ID。

### D7：R07 不可读时允许 supporting fallback

R07 URL 仍按官网访问结果分类。若它只能返回 SPA 壳，可以读取与其路径相对应的 SpacemiT 官方 GitHub 文档来建立 SDK baseline；Act Response 必须标为 `交叉验证`，并保留 R07 的 `unreachable` 结果。该 fallback 不改变 R01 唯一权威正文来源，也不允许用 GitHub 静默覆盖官网差异。

## Risks / Trade-offs

- [SpacemiT 页面继续无法读取] → 对应 URL 归类为 `unreachable`；SDK baseline 可用对应官方 GitHub 文档建立，但必须标明交叉验证等级。
- [初始 baseline 无正文快照] → 只比较 D1 的已持久化字段；新字段只建立当前 baseline，不伪造上次值。
- [来源变化涉及技术正文] → 记录受影响主题并返回 Plan；不侵入 MS03 及以后 change。
- [GitHub 页面与官网不一致] → 保留差异和解除条件，以官网为权威。
- [moved 导致覆盖行数增加] → 验证唯一性和旧行保留，不把固定 38 行误作完成条件。

## Migration Plan

1. 修订 `source-refresh.md` 的首次 baseline 比较和 SPA 壳分类规则。
2. 按固定顺序重新刷新 7 个 URL，并在每个 URL 后更新 Act Response 的已检查清单与 `next`。
3. 只按实际结果精准修改 `source-coverage.md`。
4. 仅在满足条件时修改 `known-gaps.md`；否则记录跳过原因。
5. 验证逐 URL 结果、URL 唯一性、未检查行、Markdown 和 OpenSpec change。

回退时只撤销本 change 对三个候选产品文档的实际差异；Act Response 保留失败或阻塞事实。全局状态由后续 `openspec-docs-maintainer` 根据 accepted 结果同步。

## Open Questions

None. 来源在 Act 时的实际结果属于执行输入；所有结果分支和停止条件已定义。
