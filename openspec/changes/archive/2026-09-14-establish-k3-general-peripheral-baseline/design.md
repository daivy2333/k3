## Context

见 [proposal.md](proposal.md) 的动机与批准范围。仓库目前没有 `docs/peripherals/` 正文；[总索引](../../../docs/index.md)把六个目标入口标为待聚合，[来源覆盖表](../../../docs/reference/source-coverage.md)中的对应 URL 均为 `future / deferred / partially-observed`，源端修订为 `unknown`。

当前可审计材料分为三层：K3 datasheet 提供 GPIO、PWM 和 WDT 等 SoC 能力；官方 Linux `k3-br-v1.0.y` DTS 提供 GPIO、PWM0–PWM19、两个 IR-RX、六路 AP I2S、WDT、MMIO RTC、RPMI RTC 及其静态资源；CoM260 产品资料提供 I2S、PWM/FAN、IR 和 GPIO 复用或电源引出。六个 Buildroot 操作页在当前环境无法直接取得正文，因此不能提供 K3 专属命令和运行结果。

调查发现一项实质来源冲突：K3 datasheet 概览写 30 路 PWM，详细章节写 20 路 PWM0–PWM19，当前 `k3.dtsi` 也只观察到 PWM0–PWM19。该冲突不影响三篇文档拆分，但禁止计划把任一数字提升为无条件统一结论。

## Goals / Non-Goals

**Goals:**

- 用统一链路表达 `SoC 能力 → 控制器资源 → DTS 状态 → 模组/Kit 连接 → 软件或运行边界`。
- 让控制器、pinmux/端点、DMA/IRQ 依赖、consumer 和生命周期责任可独立检索。
- 复用既有平台、中断与 DMA 基线，避免重复定义资源提供者和所有权规则。

**Non-Goals:**

- 不补写无法直接观察的 Buildroot 操作步骤、用户空间接口或真板行为。
- 不通过修改既有 MS03–MS10 正文裁决来源冲突。
- 不为文档验证新增脚本、抓取器、manifest 或 Evidence 目录。

## Decisions

### D1：三篇正文按数据路径和生命周期拆分

创建 `k3-gpio-pwm-ir.md`、`k3-audio.md` 和 `k3-wdt-rtc.md`。GPIO、PWM 与 IR-RX 共同依赖 pinctrl 和板级复用；Audio 有独立的 DAI、clock、DMA、codec/endpoint 和 stream 边界；WDT 与 RTC 都涉及持续计时及复位或 alarm 生命周期。六篇单设备文档会形成多个短小空壳，全部合并则会混合 I/O、流式 DMA 和系统生命周期三个故障域。

### D2：来源入口与首篇正文在同一 Iteration 激活

Iteration 000 把六个官方入口调整为当前 MS11 职责，同时保留 `partially-observed` 和 `unknown`；同轮建立 GPIO/PWM/IR-RX 正文。只更新来源而没有正文会留下不可导航的 active 状态，因此不拆成单独 Iteration。若 Act 能直接取得对应页，可在同一证据规则下补充；若内容改变 requirement、拆分或 Acceptance，则停止并返回 Plan。

### D3：PWM 数量冲突并列保留

正文同时记录概览的 30 路、详细章节的 20 路 PWM0–PWM19，以及 DTS 中已观察的 PWM0–PWM19；三者不互相补值。选择最新页面中的任一数字会丢失同一官方文档内部冲突，修改既有 SoC 概述也属于独立 refresh，因此均不采用。

### D4：GPIO 与 pinctrl 责任必须分离

GPIO controller 的 bank/range、读写和 IRQ 能力与 pinctrl 的 mux、电气配置分别陈述。CoM260 引脚表只证明复用候选和电压域，不证明目标 DTS 已选择该功能或外部电路允许驱动。把 pin 名直接整理为“可用 GPIO 列表”会隐藏占用和电气风险，因此不采用。

### D5：Audio 以端点链表达，不建立未证 stream 状态机

Audio 正文按 `I2S/SSPA controller → CPU DAI → sound card → codec/display endpoint` 描述静态依赖，并单列 clock/reset、DMA channel、audio power domain 与 CoM260 I2S 引脚。DMA/cache 只引用 MS06。没有 codec、route、buffer 生命周期、IRQ/xrun 或 stream 日志时，不构造启动、运行、停止和恢复状态机。

### D6：WDT、MMIO RTC 与 RPMI RTC 分开建模

WDT 只记录计数、clock/reset、IRQ 和 restart 属性边界；RTC 分为直接 MMIO 节点与通过 mailbox 暴露的 RPMI RTC。两条 RTC 静态路径同时为 `okay` 不能证明 Linux 运行所有权、时间同步或 alarm 唤醒关系，VCC_RTC 也不能证明掉电保持已经验证。

### D7：验证直接检查知识产物

测试见证使用目标文件缺失、六个来源仍 deferred、索引无入口等当前 RED。GREEN 检查首行来源、必需章节、证据等级、四字段未知项、相对链接、URL 唯一、术语与计数一致，以及 OpenSpec 严格校验。验证结果写入 Act Response，Persisted Evidence 为 `none`。

## Risks / Trade-offs

- [Buildroot 入口不可达] → 保留权威 URL 和访问边界，仅使用已核对的官方 supporting source，不编造操作行为。
- [PWM 数量自相冲突] → 三种观察并列；未来 source refresh 再裁决全局基线。
- [CoM260 DTS 变体不唯一] → 每项资源标明来源层级，沿用 G7，不选择默认板型。
- [Audio 静态资源较多] → 保持独立正文；超过 M02 的 500 行建议上限时在同一 requirement 内继续拆分。
- [WDT/RTC 验证可能改变系统状态] → 本 change 只做文档静态验证，不规划 reset、alarm 或 suspend 测试。

## Migration Plan

按 Iteration 依次建立来源与 GPIO/PWM/IR-RX、Audio、WDT/RTC，最后统一更新缺口、术语和索引。任一 Iteration 未通过 Review 时不展开下一轮。回滚只需撤销本 change 新增或修改的 Markdown；不涉及数据迁移或运行状态。
