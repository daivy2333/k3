# K3 CoM260 串口启动流程

- Status: active
- Last validated: 2026-09-11
- Environment: K3 CoM260 Kit (SpacemiT K3 SoC); DC 12V/6A 电源适配器; USB-TTL 3.3V 串口模块; 3 根杜邦线 (GND/TX/RX); PC 串口终端 (MobaXterm / PuTTY / Tera Term); 115200-8-N-1 无流控
- Source: 本次会话 (2026-09-11) 用户在 K3 CoM260 Kit 上实跑串口启动观察, 观察到从 BootROM 到 Bianbu Linux 4.0.1 root Shell 的完整流程

## 适用范围

**适用**:

- K3 CoM260 Kit 出厂预装 Bianbu Linux 4.0.1 镜像的首次上电与串口启动观察
- 通过 USB-TTL 串口验证 CoM260 Kit 板级供电、UART 物理层、bootloader 与 Linux 用户空间的串口链路
- 现场 bring-up、回归测试、固件升级后的基线观察

**不适用**:

- download boot (BROM-Fastboot / U-Boot Fastboot / ADB bootloader) 刷机流程
- K3 其它板卡 (Pico-ITX / K1 等) 的启动观察
- 启动后业务验证、功耗测试、性能 benchmark
- 自编译镜像或其它发行版的启动流程 (默认 root 密码可能不同)

## 前置条件

**硬件**:

- K3 CoM260 开发板
- DC 电源适配器, 建议 `12V 6A`
- USB 转 TTL 串口模块 (3.3V 电平, CH340 / CP2102 / FT232 等)
- 3 根杜邦线 (GND / TX / RX 各一根)
- PC 一台 (用于连接 USB-TTL 和运行串口终端)

**软件**:

- 串口终端软件, 例如:
  - Windows: MobaXterm / PuTTY / Tera Term
  - Linux: minicom / picocom / screen
  - macOS: minicom / screen / CoolTerm

**人员**:

- 知晓如何安全接入 DC 电源
- 知晓如何配置 PC 串口终端

## 操作步骤

### 步骤 1: 连接 USB-TTL 串口 (只接 3 根线)

按下列对应关系连接 K3 CoM260 Kit 与 USB-TTL 模块 (TX/RX 必须交叉):

| K3 端 | USB-TTL 端 |
|---|---|
| TX | RX |
| RX | TX |
| GND | GND |

**警告**: 不要接 USB-TTL 的 `VCC` / `5V` 引脚到 K3。K3 由 DC 电源单独供电, USB-TTL 只承担信号传输, 不向 K3 供电; 接错电平可能损坏器件。

### 步骤 2: 配置串口终端

在 PC 上打开串口终端, 按下列参数配置:

| 参数 | 值 |
|---|---|
| Baud rate | 115200 |
| Data bits | 8 |
| Stop bits | 1 |
| Parity | None |
| Flow ctrl | None |

即常说的 `115200 8N1`。

### 步骤 3: 上电并观察

按下列顺序操作:

1. USB-TTL 接电脑
2. 打开串口终端
3. 设置 115200 8N1
4. 给 K3 接 DC 电源
5. K3 自动上电启动
6. 串口开始输出日志

### 步骤 4: 观察典型启动顺序

串口输出应按下列阶段顺序出现, 每个阶段有可识别的标志字段:

| 阶段 | 典型输出标志 |
|---|---|
| 1. BootROM | 早期 BROM 阶段日志 |
| 2. SPL / FSBL | 1 级引导阶段日志 |
| 3. OpenSBI | `OpenSBI v...`; `Platform Name : SpacemiT K3`; `Platform HART Count : ...` |
| 4. U-Boot | `U-Boot ...`; `DRAM: ...`; `Core: ...`; `MMC: ...`; `Loading Environment ...` |
| 5. Linux Kernel | `Loading kernel...`; `Loading device tree...`; `Starting kernel ...` |
| 6. Kernel 初始化 | `[    0.000000] Linux version ...`; `[    0.xxxxxx] Machine model: SpacemiT K3 ...` |
| 7. rootfs 挂载 | rootfs 挂载相关内核日志 |
| 8. systemd / init | `systemd[1]: System initialization...`; `systemd[1]: Started ...` |
| 9. Bianbu Linux 启动 | Bianbu 4.0.1 banner |
| 10. ttyS0 登录界面 | `k3 login:` |

完整启动链 (按用户观察):

```text
通电
 ↓
BootROM
 ↓
SPL / FSBL
 ↓
OpenSBI
 ↓
U-Boot
 ↓
Linux Kernel
 ↓
rootfs
 ↓
systemd
 ↓
Bianbu Linux
 ↓
ttyS0 登录界面
```

## 验证

### 成功判据 1 — 系统启动完成

串口终端最终稳定显示下列 banner 与登录提示:

```text
Bianbu 4.0.1 k3 ttyS0

k3 login:
```

看到这两行说明 Bianbu Linux 已经成功启动。

### 成功判据 2 — 进入 root Shell

在 `k3 login:` 提示下输入:

```text
login: root
Password: bianbu
```

- 账号: `root`
- 密码: `bianbu` (**仅适用于官方预装 Bianbu 4.0.1 镜像**; 自编译镜像或其它发行版密码可能不同)

如果该镜像允许 root 登录并验证成功, 串口会显示:

```text
Welcome to Bianbu 4.0.1 ...

root@k3:~#
```

**最终成功**: 看到 `root@k3:~#` 提示符, 表示 K3 已经通过串口完整启动成功, 并进入 root Shell。

### 安全注意

默认 root 密码 `bianbu` 是出厂公开值, 任何拿到镜像的人都知道; 进入 root Shell 后应**立即修改密码**, 避免设备被未授权访问。修改方法示例 (在 root Shell 中执行):

```text
passwd root
```

按提示输入新密码两次即可, 提示 `passwd: password updated successfully` 即生效。

## 失败处理

| 现象 | 诊断方向 | 停止条件 |
|---|---|---|
| 串口无任何输出 | TX/RX 接反、串口参数不匹配、串口线缆损坏、Kit 未上电 | 重新检查接线、参数、上电状态; 仍无输出则停止本 runbook |
| 输出停留在 BootROM, 不进入 FSBL | 启动介质无有效镜像、boot pin 配置与介质不匹配 | 停止本 runbook, 转刷机或启动介质诊断 |
| 输出停留在 OpenSBI 或 U-Boot, 不加载 kernel | 启动介质 kernel 镜像损坏、加载地址错误 | 停止本 runbook, 转镜像完整性或 DTS 诊断 |
| `kernel panic` 或 rootfs 挂载失败 | rootfs 损坏、fstab 配置错误 | 停止本 runbook, 转镜像或 rootfs 诊断 |
| systemd 启动失败 | 用户空间服务配置错误、关键服务崩溃 | 记录 `systemctl status` 输出, 停止本 runbook |
| 长时间不出现 `k3 login:` | 用户空间服务卡死、getty 未启动 | 记录最近一次日志, 停止本 runbook |
| `login: root` + `bianbu` 密码错误 | 镜像非官方预装 Bianbu / 已被修改过密码 | 停止本 runbook, 询问镜像来源; **不要盲试默认密码** |

## 回滚

本 runbook 是纯串口观察流程, 不修改任何 K3 镜像、bootloader、UFS / eMMC / SPI-NOR 数据, 也不写入任何持久状态 (root 密码修改不属于本 runbook 范围, 应在新 runbook 中独立记录)。

- 无需回滚
- 如需重新观察, 直接断电 → 重新上电即可, 不会留下残留状态

## 证据

- 串口参数 (`115200 8N1`, 无流控) 来自用户实操记录
- USB-TTL 接线关系 (TX/RX 交叉, 共地, **不接 VCC/5V**) 来自用户实操记录
- 电源适配器规格 (`12V 6A`) 来自用户实操建议
- 启动阶段顺序 (BootROM → SPL/FSBL → OpenSBI → U-Boot → Linux Kernel → rootfs → systemd → Bianbu Linux → ttyS0 登录界面) 来自用户实跑观察
- 发行版标识 `Bianbu 4.0.1` 来自用户实跑观察的 systemd 启动后系统 banner
- 登录 console `ttyS0` 与主机名 `k3` 来自用户实跑观察的登录提示
- 默认 root 账号 `root` / 密码 `bianbu` 来自用户对官方预装 Bianbu 4.0.1 镜像的说明

## 已知范围缺口

- 本 runbook 仅覆盖 K3 CoM260 Kit 预装 Bianbu Linux 4.0.1 镜像的 local boot 路径, 不覆盖:
  - download boot / BROM-Fastboot / U-Boot Fastboot 刷机流程
  - K3 其它板卡 (Pico-ITX / K1 等) 的启动观察
  - 自编译镜像或非 Bianbu 发行版的启动流程
- 本 runbook 不展开登录后操作 (网络配置、存储介质写测试、业务验证)
- 默认 root 密码 `bianbu` 仅适用于官方预装 Bianbu 4.0.1 镜像; 自编译或定制镜像的密码由制作者设定, 密码错误时不应盲试
- 启动阶段的部分细节 (handoff 寄存器、bootinfo 配置、镜像装载地址) 不在本观察流程范围内
