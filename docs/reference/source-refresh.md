> 来源: https://www.spacemit.com/community/document/info?lang=zh&nodepath=hardware/key_stone/k3/k3_docs（源端修订: unknown；观察日期: 2026-09-02）

# 人工来源刷新指南

> 适用范围: 维护者按本指南对 [`source-coverage.md`](source-coverage.md) 中的 URL 执行一次人工复核。
> 流程规则来源: M01（单一权威源）、M02（输出形态）、M04（无可执行代码）、D01（纯 Markdown 聚合）、D02（主题驱动目录）、D03（首行格式）、D05（刷新结果与长期状态分离）。
> 状态基础: 初始 `2026-09-02` 观察仅建立 baseline，不构成一次真实 refresh；首次真实 refresh 由后续来源变更 change 完成。
> 产品文档通过获批 OpenSpec change 修改；change accepted 后，docs-maintainer 仅按实际结果同步 SNAPSHOT、tasks、M/D/K/R/I 中适用的状态。

## 目录

- [三个独立维度](#三个独立维度)
- [五种刷新结果](#五种刷新结果)
- [覆盖表与刷新结果的关系](#覆盖表与刷新结果的关系)
- [操作顺序](#操作顺序)
- [变更与缺口边界](#变更与缺口边界)
- [中断恢复](#中断恢复)
- [三种文字演练](#三种文字演练)
- [停止条件](#停止条件)
- [反例](#反例)

## 三个独立维度

人工刷新与长期覆盖是**三个独立维度**，互不替代。混淆任意两个都会产生错误结论。

| 维度 | 持久位置 | 维护者 | 何时更新 |
| --- | --- | --- | --- |
| 聚合状态 | `source-coverage.md` 的 `聚合状态` 列 | 获批 change | 主题职责、范围或优先级变化 |
| 访问状态 | `source-coverage.md` 的 `访问状态` 列 | refresh change | 本次复核取得正文或只能确认目录 |
| 刷新结果 | refresh change 的 Act Response | refresh change | 一次复核对单个 URL 给出唯一结论 |

刷新结果**不写入** `source-coverage.md` 任何列。覆盖表不增加 `本次结论` 列；一次检查的结论只活在执行它的 refresh change 的 Act Response 里。

## 五种刷新结果

一次人工刷新检查对单个 URL 只能给出以下五种结果之一；五种结果互斥。

### `unchanged`

- **判定**: 源端修订日期未变，正文相关章节未变。
- **动作**: 只允许更新该 URL 行的 `观察日期` 字段；不改写其他字段，不触发任何主题文档。
- **记录位置**: refresh change 的 Act Response 中该 URL 对应行的 `结论: unchanged` 一项。
- **不变量**: 覆盖表的 `源端修订`、`访问状态`、备注均不改变。

### `changed`

- **判定**: 源端修订日期变化，或正文相关章节实质更新。
- **动作**: 分两阶段处理。预检查（change 外）只把该 URL 归为 `changed` 并在草稿中记录受影响主题，不修改任何产品文档；随后创建独立 refresh change，获批后由 Act 在该 change 批准的范围内更新受影响正文与覆盖表行的 `源端修订`、`观察日期` 与必要的 `备注`。
- **记录位置**: 预检查的逐 URL 结论与「已检查清单」写入 refresh change 的 Act Response；正文与覆盖表更新由同一获批 change 完成。
- **不变量**: 若获批 change 执行中发现超出其批准范围的 `changed`/`moved`/`removed`，停止并返回 Plan，不创建含糊的「父 change / 新 change」链。

### `moved`

- **判定**: 源页面 URL 改变；可由 R01 入口或重定向定位到新地址。
- **动作**: 在覆盖表追加新 URL 行并填写完整字段；旧 URL 行保留，其 `聚合状态` 与 `访问状态` 保持不变，只在 `备注` 注明「已迁移至 <新 URL>」；迁移事实写入 refresh change 的 Act Response。是否登记 `known-gaps.md` 缺口由替代来源是否已确认决定：R01 入口或重定向已确认新 URL 的，替代来源已确认，不存在待解除缺口，不得登记 `known-gaps.md`；只有疑似移动但从 R01 找不到替代来源时，才经获批 change 登记 `known-gaps.md` 缺口。不得删除旧 URL，不得用 sed/inline 替换，不得把旧来源改成 `out-of-scope`。
- **记录位置**: refresh change 的 Act Response 中 `moved: <旧 URL> → <新 URL>` 一行；针对新行，如暂未取得正文，`访问状态` 先记 `partially-observed`。
- **不变量**: 旧 URL 行及其聚合/访问状态不被改写；任何引用旧 URL 的主题文档 `> 来源:` 行在迁移确认后由独立 change 改写，本 change 不擅自改写；已确认替代来源的 `moved` 不留下任何 `known-gaps.md` 缺口条目。

### `removed`

- **判定**: 官方入口或显式响应表明页面已永久下架；非暂时不可访问。
- **动作**: 在覆盖表保留旧 URL 行，其 `聚合状态` 与 `访问状态` 保持不变，只在 `备注` 注明永久下架事实；把永久下架导致的来源缺失登记为 `known-gaps.md` 缺口；不得删除旧 URL，不得把旧来源改成 `out-of-scope`，不得静默改用其他 URL 替代。
- **记录位置**: refresh change 的 Act Response 中 `removed: <URL>` 一行；`known-gaps.md` 包含对应 `## G<idx>.` 段落（经获批 change 修改）。
- **不变量**: 旧 URL 行及其聚合/访问状态不被改写；不得将暂时不可访问判为 `removed`。

### `unreachable`

- **判定**: 当前网络或访问条件下无法取得正文；既无 `changed` 的证据也无 `removed` 的官方信号。
- **动作**: 预检查（change 外）只在该 URL 对应的草稿中记录 `unreachable` 结论，不修改任何产品文档；获批 refresh change 内可更新已检查行的 `观察日期`（记本次访问时间），并在有证据支持时更新 `访问状态`；保留 URL，按 R01 入口或官方 GitHub 再次尝试。长期不可访问且从 R01 找不到替代来源时，通过获批 change 登记 `known-gaps.md` 缺口。
- **记录位置**: refresh change 的 Act Response 中 `unreachable: <URL>` 一行；长期不可访问且无替代来源时，同时登记对应 `known-gaps.md` 缺口（经获批 change）。
- **不变量**: 不得把暂时不可访问判为 `removed`，不得静默改用其他 URL 替代；多次 `unreachable` 累计不得触发覆盖表行的 `聚合状态` 改变，长期不可访问的升级由独立 change 显式决定。

## 覆盖表与刷新结果的关系

`source-coverage.md` 是长期状态的唯一所有者；refresh change 是刷新结果与行级元数据的记录位置。两者职责严格分离。

| 行为 | 覆盖表 | refresh change Act Response | 主题文档 |
| --- | --- | --- | --- |
| 长期 `聚合状态` 变化 | 修改 | 引用 change 名 | 由独立 change 修改 |
| 长期 `访问状态` 变化 | 修改（refresh change 内） | 列出每行变化 | 不动 |
| `观察日期` 更新 | 修改（refresh change 内） | 列出每行变化 | 不动 |
| 一次检查的 `刷新结果` | **不动** | 唯一记录位置 | 不动 |
| `moved`/`removed` 的旧 URL | 保留行，聚合/访问状态不动，`备注` 记迁移或下架 | `moved: <旧> → <新>` / `removed: <URL>` | 由独立 change 改 |
| `removed` 或长期不可访问后的缺口 | `备注` 简述 | 经获批 change 引用 `known-gaps.md` | 不动 |

## 操作顺序

按以下顺序执行一次 refresh；任一步骤失败或结果异常都先记录再决定继续或返回。

1. **选择范围**: 决定本次检查的 URL 集合（建议一次不超过 10 个 URL）；明确是覆盖全表还是仅限 `active` 行。
2. **读取来源**: 浏览器或 curl 打开每个 URL；记录源端修订日期、标题与上次 refresh 相比是否变化。
3. **比较修订与正文**: 与 `source-coverage.md` 当前行的 `源端修订` 和 `访问状态` 对比；把差异归类到五种结果之一。
4. **行级更新（草稿）**: 在草稿中列出每个 URL 的新 `观察日期`、`访问状态`、`备注` 变化；预检查阶段不修改覆盖表，仅形成逐 URL 结论。一个 refresh change 一次可覆盖多个 URL，但 Act Response 必须对每个 URL 单独记录结果与「已检查清单」，不得以「全部 unchanged」等聚合结论替代。
5. **按需创建 change / 两阶段边界**: 预检查只将结果分类并提案——任何会修改产品文档的 URL 结果（`unchanged` 仅更新 `观察日期`、`unreachable` 仅更新 `观察日期` 与有证据的 `访问状态`、`changed` 更新正文与覆盖表、`moved` 追加新行、`removed` 登记缺口或长期 `unreachable` 无替代来源时登记缺口）都必须先有获批的 refresh change；预检查本身永远不修改 `source-coverage.md` 任一列。Act Response 只登记逐 URL 结论与指向。产品文档（覆盖表、正文、缺口）的更新只在对应 refresh change 获批后由 Act 按批准范围完成；批准范围内执行时若发现新的超出范围的 `changed`/`moved`/`removed`，停止并返回 Plan。
6. **更新正文**: refresh change 经 Plan 与 Act 通过后，由 Act 修改对应主题文档 `> 来源:` 行或事实归属；本指南不直接执行。
7. **验证**: 运行全量 Markdown 链接解析、OpenSpec validate、coverage 行数与唯一性检查；如出现新缺口则同步 `known-gaps.md`。

## 变更与缺口边界

refresh 与缺口登记是两种不同的处理路径，由结果类型决定走哪一条。

- **`changed` → refresh change（两阶段）**: 预检查只分类并提案，不修改产品文档；获批 refresh change 才更新受影响正文与覆盖表行。
- **`moved` → 保留旧 URL + 新 URL 行**: 旧行保留且聚合/访问状态不变，迁移事实写入旧行 `备注` 与 Act Response；R01 入口或重定向已确认新 URL 时不登记 `known-gaps.md`，只有疑似移动且无替代来源时经获批 change 登记缺口；新行由 refresh change Act 写入。
- **`removed` → 保留旧 URL + 缺口登记**: 旧行保留且聚合/访问状态不变，下架事实写入旧行 `备注`、Act Response 与 `known-gaps.md` 获批修改。
- **`unreachable` → 预检查草稿暂存**: change 外不写覆盖表也不写缺口；获批 refresh change 内可更新已检查行的观察日期/访问状态；长期不可访问且从 R01 无替代来源时经获批 change 登记缺口。

任何 `> 来源:` 行的真实变化都必须经获批 change 改写；本仓库不接受在 refresh 过程中以 git diff 静默修改。

## 中断恢复

人工刷新可中断于任何阶段，恢复时不丢失已完成工作。

- **已完成行**: 由 refresh change 草稿或 Act Response 中的「已检查清单」记录；草稿可在多轮之间持久化在同一 change 的工作区。
- **未检查行**: 覆盖表对应行的 `观察日期`、`访问状态`、`源端修订`、`备注` 保持上次值不变；不预设为 `unchanged`。
- **恢复点**: 由「已检查清单」与覆盖表当前行内容决定；具体下一条 URL 由草稿或 Act Response 中 `next: <URL>` 字段给出。
- **完成语义**: 部分完成永远只更新已检查行；中断时保持 Cycle 未完成。一般续跑保持 `pending`，真实阻塞使用 `blocked` handoff。Act 把 Act Response 改为 `reported` 只表示等待 Plan Review；只有 Plan Review 给出 `accepted` 才表示 Cycle/Iteration 验收完成。

## 三种文字演练

以下演练不修改任何文件，仅用于验证本指南的判定与恢复规则；可逐步执行。

### 演练 1：unchanged

初始: `source-coverage.md` 现有 38 行；`观察日期` 全部为 `2026-09-02`。

1. 选取 R05 的 5 个 `active` URL；
2. 浏览器逐个打开，确认源端修订未变化，正文相关章节与上次观察一致；
3. 草稿中列出这 5 行的 `观察日期` → `2026-09-04`，`访问状态` 不变；
4. 创建 refresh change `refresh-r05-2026-09-04`，Plan Context 仅允许行级 `观察日期` 更新与 Act Response 登记；
5. Act Response 逐 URL 写入 `unchanged: <URL1>` … `unchanged: <URL5>` 并附「已检查清单」，覆盖表对应 5 行 `观察日期` 改为 `2026-09-04`；其余 33 行保持 `2026-09-02`；
6. 验证: 38 行总数不变，访问状态分布不变，主题文档无修改。

期望结果: 5 行更新；其他 33 行原值不变；无主题文档 `> 来源:` 变化。

### 演练 2：unreachable

初始: 与演练 1 相同。

1. 选取 R04 中 1 个 URL `k3_ds.md`，浏览器打开超时或返回 5xx；
2. 预检查阶段不修改任何产品文档：覆盖表 `聚合状态` 与 `访问状态`、`known-gaps.md` 均不动；
3. 草稿记录 `unreachable: k3_ds.md @ 2026-09-04`；
4. refresh change 的 Plan Context 仅允许将该行 `观察日期` 改为 `2026-09-04`；既有 `访问状态` 为 `partially-observed` 保持不变，因为单次超时/5xx 不能抹除已有官方目录或检索片段证据；只有新证据（页面被官方显式下架、官方目录移除该条目、或确认原访问方法永久失效）证明原访问证据不成立时，才能由获批 change 决定将 `访问状态` 改为 `unverified`；
5. Act Response 写入该结论；不创建缺口登记；不修改主题文档；
6. 后续若重新可访问，由下一次 refresh change 决定升回 `partially-observed` 或 `observed`。

期望结果: 1 行 `观察日期` 更新；既有 `访问状态` 保持不变；无 `removed`、无缺口、无静默替换；不存在仅凭网络失败就降级访问状态的场景。

### 演练 3：partial-resume

初始: refresh change `refresh-r05-2026-09-04` 草稿已批准但 Act 仅完成 2 个 URL。

1. 中断时 Act Response 写入「已检查清单: <URL1>, <URL2>」与 `next: <URL3>`；
2. 覆盖表只有 `<URL1>`、`<URL2>` 的 `观察日期` 更新为 `2026-09-04`；其他 36 行不变；
3. 维护者恢复时，读取 Act Response 中 `next: <URL3>` 继续；
4. 继续完成 3 个 URL 后再次中断，`next: <URL6>`；
5. 恢复后继续直到 5 个 URL 全部检查；Act Response 状态由 `pending` 改 `reported`（仅表示等待 Plan Review，仍需 Review `accepted` 才算验收通过）；
6. 整个过程不修改主题文档，不重写已检查行，不把未检查行标为 `unchanged`。

期望结果: 5 行逐步更新；中断处可任意续跑；未检查行在恢复前保持原值。

## 停止条件

以下任意条件命中即停止当前 refresh 并返回 Plan，不进入第四次尝试。

- 发现真实来源变化但 Plan Context 未包含相应处理。
- references URL 集合在执行前后不再是 38（BASELINE-CHANGED）。
- `moved` 后的新 URL 在 R01 中无法定位。
- 任意 `removed` 结论缺乏官方信号或显式响应。
- 同一 URL 连续 3 次 `unreachable` 且无替代来源信号。
- 当前 refresh change 需要改变本 change 已 accept 的 requirement、scenario 或 design。

## 反例

以下做法违反本指南，禁止使用：

- 把 `unchanged` / `changed` 等结论直接写入 `source-coverage.md` 任一列。
- 用 git diff 或编辑工具在 refresh 之外静默修改主题 `> 来源:` 行。
- 把 `unreachable` 与 `removed` 混为一谈；把「暂时不可访问」判为 `removed`。
- 把 `moved` 或 `removed` 后的旧 URL 直接删除，或改写旧行的聚合/访问状态、把旧来源改成 `out-of-scope`。
- 在获批 refresh change 执行中发现超出批准范围的 `changed`/`moved`/`removed` 时继续推进，而不是停止并返回 Plan。
- 在 refresh change 内同时实现主题正文改写与覆盖表行级更新，而不区分两者的 change 边界。
- 创建运行 ID、manifest、hash 账本或 Evidence 目录来证明一次 refresh 存在。
- 把初始 `2026-09-02` 观察伪装为一次真实 refresh。
- 一个 refresh change 覆盖多个 URL 时，用单个「全部 unchanged」聚合结论替代逐 URL 记录（Act Response 必须保留每个 URL 的结果与「已检查清单」）。
