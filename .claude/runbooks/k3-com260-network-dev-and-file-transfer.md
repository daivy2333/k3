# K3 CoM260 网络开发与文件传输流程 (草稿, 未实跑验证)

- Status: **draft / unverified**
- Last validated: N/A (未在真板实跑)
- Environment (预期, 未验证): K3 CoM260 Kit (SpacemiT K3 SoC) + Windows PC; RJ45 网线; 同一二层广播域; SSH/SCP/SFTP 客户端 (Windows: OpenSSH / MobaXterm / PuTTY); 备选用 U 盘
- Source: 本次会话 (2026-09-11) 用户提供的典型网络开发与文件传输流程描述; **所有具体动作、IP 网段、U 盘分区路径 (`/dev/sda1`)、`scp` 示例命令均未在真板上执行或采集**, 仅为典型流程整理, 用于指导首次实跑。

## 适用范围 (预期, 待实跑后修订)

**适用 (预期)**:

- K3 CoM260 Kit 完成 R18 串口启动观察后, 通过以太网与 PC 互联, 进行日常开发、远程控制、文件传输、编译测试
- 以太网直连场景 (无交换机 / 路由器)
- U 盘作为离线备用文件通道

**不适用 (预期)**:

- 通过路由器 / 交换机 / VLAN 划分的复杂网络环境
- 无线网络 (K3 CoM260 Kit 不带 Wi-Fi 模块, 需 USB Wi-Fi dongle 另议)
- 跨网段路由场景
- 嵌入式网络协议栈的深入配置 (防火墙 / iptables / IPv6)
- VS Code Remote / IDE 集成等具体开发工具的接入 (本 runbook 不展开)

## 前置条件 (预期, 待实跑后修订)

**硬件 (预期)**:

- K3 CoM260 Kit (已经通过 R18 完成串口启动, root Shell 可用)
- Windows PC 一台 (具备 RJ45 网口, 或 USB-Ethernet 适配器)
- RJ45 网线一根 (直通线, 通常为 Cat5e / Cat6)
- 可选: U 盘一个 (FAT32 / exFAT 格式, ≤ 128 GB)

**软件 (预期)**:

- Windows 自带 OpenSSH 客户端 (Windows 10 1809+ / Windows 11 默认安装)
- 或 MobaXterm / PuTTY (SSH 客户端)
- K3 端 SSH Server (典型发行版默认安装, 启动后由 systemd 拉起 sshd.service)
- 可选: rsync (Windows 端可通过 WSL / MSYS2 / Git Bash 获取)
- 可选: 串口终端 (用于初始网络配置, 见 R18)

**人员 (预期)**:

- 知晓 K3 CoM260 Kit 的 root 凭据 (见 R18)
- 知晓 Windows 网络设置 (IP / 子网掩码 / 网关 / 共享)

⚠️ **采集缺口**: 上述前置条件中, **K3 端 `sshd.service` 是否默认启动、Windows 默认防火墙是否放行 SSH、K3 默认 IP 段是什么**, 均未在真板验证。

## 操作步骤 (预期, 待实跑后修订)

### 步骤 1: 网线直连 K3 和 PC

⚠️ **未采集**: 用户描述「Windows PC 用 Ethernet 直连 K3 CoM260」, 但实际接线、网卡识别、链路指示灯状态、网口协商速率 (10/100/1000 Mbps) 均未观察到。

预期操作 (待实跑修订):

1. 将 RJ45 网线一端插入 K3 CoM260 板载 / 底板网口 (典型为 GMAC 控制器, 详见 R16)
2. 另一端插入 PC 网口
3. 等待两端网卡 link 指示灯亮起 (1 Gbps / 100 Mbps)

```text
Windows PC
    │
    │ Ethernet
    ▼
K3 CoM260
```

### 步骤 2: 建立同网段 IP

⚠️ **未采集**: 用户描述「只要 Windows 和 K3 获得同一网段的 IP, 就可以互相通信」, 但**具体网段、IP 分配方式 (静态 / DHCP)、K3 默认 IP 是什么**, 均未验证。

**方案 A: 静态 IP (预期, 待实跑修订)**

预期操作:

- Windows 端: 设置静态 IP, 例如 `192.168.10.1/24` (⚠️ 仅为示例, 待真板验证)
- K3 端: 设置静态 IP, 例如 `192.168.10.2/24`, 子网掩码 `255.255.255.0`, 网关留空 (⚠️ 仅为示例, 待真板验证)
- 在两端互 `ping` 验证链路

**方案 B: Internet Connection Sharing (ICS) (预期, 待实跑修订)**

预期操作:

- Windows 通过 Wi-Fi 上网, 同时通过 Ethernet 接 K3
- 开启 ICS (Internet Connection Sharing) 把 Wi-Fi 共享给 Ethernet 适配器
- K3 通过 Ethernet 经 Windows 访问互联网

```text
Internet
   │
 Wi-Fi
   ▼
Windows
   │
Ethernet
   ▼
K3
```

### 步骤 3: SSH 远程登录

⚠️ **未采集**: 用户描述「K3 开启 SSH Server 后, Windows 可以直接 `ssh root@K3_IP`」, 但 **K3 端 sshd.service 状态、首次启动 sshd 的命令、防火墙规则、root 登录是否被 PermitRootLogin 允许**, 均未验证。

预期操作:

- 在 K3 上检查 sshd 状态: `systemctl status sshd` (⚠️ 未实跑, 输出待真板验证)
- 在 Windows 上执行: `ssh root@<K3_IP>` (⚠️ 未实跑, `<K3_IP>` 占位符待真板验证)
- 成功后, Windows 通过 SSH 进入 K3 的 root Shell

```text
Windows
   │
   │ SSH
   ▼
K3 Linux
```

### 步骤 4: 通过网络传输文件

⚠️ **未采集**: 用户提供了 `scp` 示例命令, 但**未在真板执行**; 路径 `/root/` 是否存在、权限、磁盘空间均未验证。

预期 Windows → K3 命令:

```powershell
scp test.txt root@<K3_IP>:/root/        # ⚠️ 未实跑, <K3_IP> 占位符待真板验证
```

预期 K3 → Windows 命令:

```powershell
scp root@<K3_IP>:/root/result.txt .     # ⚠️ 未实跑, <K3_IP> 占位符待真板验证
```

可用工具 (预期):

- `scp` (OpenSSH 自带)
- `sftp` (OpenSSH 自带, 交互式)
- `rsync` (需安装, 支持增量同步)

预期可承担的任务:

```text
远程控制
文件传输
程序运行
编译测试
日志获取
```

### 步骤 5 (备用): U 盘文件传输

⚠️ **未采集**: 用户描述的 U 盘流程, 包括 `lsblk` 实际输出、U 盘是否被 K3 识别为 `/dev/sda1`、挂载命令是否成功, **均未验证**。`/dev/sda1` 仅为典型示例路径, **不代表实际 K3 上的设备节点**; 实际可能是 `sda1` / `sdb1` / `mmcblk0p1` 等, 取决于 K3 USB 控制器与 U 盘枚举顺序。

预期操作 (举例, 待实跑修订):

1. 在 Windows 上把文件复制到 U 盘
2. 插 U 盘到 K3 CoM260 Kit 的 USB-A 口
3. K3 端查看块设备: `lsblk` (⚠️ 未实跑, 实际设备节点待真板验证)
4. 假设 U 盘分区是 `/dev/sda1` (⚠️ **仅为示例, 实际设备节点待真板验证**)
5. 挂载 (⚠️ 未实跑):

    ```bash
    mkdir -p /mnt/usb
    mount /dev/sda1 /mnt/usb           # ⚠️ /dev/sda1 为示例路径, 待真板验证
    ls /mnt/usb
    ```

6. 复制文件: `cp /mnt/usb/test.txt /root/`
7. 卸载: `umount /mnt/usb`

```text
Windows
   ↓
  U 盘
   ↓
  K3
```

## 最终开发结构 (预期, 待实跑后修订)

```text
                         K3 CoM260
                             │
                         Ethernet
                             │
                    SSH / SCP / SFTP
                             │
                         Windows
                         /      \
                      Wi-Fi      IDE
                        │         │
                    Internet   后续接入
                               VS Code 等

备用通道：

Windows ←→ U盘 ←→ K3
```

浓缩流程 (预期, 待实跑后修订):

```text
网线连接 K3 和电脑
        ↓
建立同网段 IP
        ↓
SSH 远程登录
        ↓
SCP / SFTP 传输文件
        ↓
远程编译、运行、测试
        ↓
后续再接 VS Code 等开发工具

备用：
U盘进行离线文件传输
```

## 验证 (待实跑后填写)

⚠️ **未采集**: 用户没有提供真板实跑证据, 以下验证项均为**预期成功判据, 实际成功判据需真板执行后填入**。

预期成功判据 (待实跑修订):

- [ ] PC 与 K3 之间能 `ping` 通 (需要同网段, 双向 ICMP)
- [ ] PC 能 `ssh root@<K3_IP>` 登录 K3, 不需要密码或能通过 R18 默认密码登录
- [ ] `scp` 双向文件传输成功
- [ ] (可选) K3 通过 Windows ICS 访问互联网
- [ ] (备用) U 盘被 K3 识别并可挂载, 文件可读写

## 失败处理 (待实跑后填写)

⚠️ **未采集**: 用户没有提供真板实跑失败现象, 以下为**典型失败模式, 实际处理方式需真板执行后填入**。

| 预期现象 | 预期诊断方向 | 预期停止条件 |
|---|---|---|
| 网线 link 灯不亮 | 网线 / 网口 / 网卡 物理故障 | 更换网线或网口 |
| `ping` 不通 | IP 不在同一网段、子网掩码错误、防火墙拦截 | 重新配置 IP, 检查防火墙 |
| SSH 连接被拒绝 | K3 sshd 未启动、K3 防火墙拦截、root 登录被禁用 | 检查 `systemctl status sshd`、防火墙、`/etc/ssh/sshd_config` |
| `Permission denied` | 密码错误 (R18 默认密码 `bianbu` 仅适用于官方预装镜像) | 见 R18 |
| U 盘未识别 | USB 端口故障、U 盘格式不兼容 (NTFS 需内核 ntfs-3g 驱动) | 换 USB 口, 检查 `dmesg` 输出 |
| 挂载失败 | 设备节点错误、文件系统错误 | 确认设备节点 (`lsblk` / `fdisk -l`), 检查 `dmesg` |

## 回滚

本 runbook 不修改 K3 镜像、bootloader、U 盘数据, 也不在 K3 上写入持久状态 (除 `mkdir /mnt/usb` 这种临时目录外)。

- 无需回滚
- 拔掉 U 盘前 `umount` 即可, 不会留下残留状态

## 证据 (采集缺口)

⚠️ **本 runbook 所有具体动作、IP 网段、U 盘设备节点 (`/dev/sda1`)、`scp` 示例命令均未在真板执行或采集**, 仅为用户提供的典型网络开发流程描述, 用于指导首次实跑。

采集缺口清单 (待真板实跑后补齐):

- [ ] K3 默认 IP 段 / 静态 IP 配置方法
- [ ] K3 sshd.service 状态 (是否默认启动 / 启动命令)
- [ ] Windows 防火墙对 SSH 的放行规则
- [ ] K3 GMAC 网口速率、双工协商结果 (见 R16)
- [ ] U 盘在 K3 上的实际设备节点 (`lsblk` 输出)
- [ ] K3 文件系统对 NTFS / exFAT / FAT32 的支持情况
- [ ] `scp` 双向文件传输的实测命令与输出
- [ ] (可选) Windows ICS 共享网络后 K3 访问互联网的实测

升级路径: 当上述缺口全部或关键项有实跑证据后, 本 runbook 可由 `Status: draft` 升级为 `Status: active`, 证据字段填写实测结果, 各节 `⚠️ 未采集` / `⚠️ 未实跑` / `⚠️ 为示例` 标记替换为实测值。

## 已知范围缺口

- 本 runbook 不覆盖:
  - 路由器 / 交换机 / VLAN 划分的复杂网络环境
  - 无线网络 (K3 CoM260 Kit 不带 Wi-Fi 模块)
  - 跨网段路由
  - 防火墙 / iptables / IPv6 深入配置
  - VS Code Remote / IDE 集成等具体开发工具接入
- 本 runbook 不展开 R18 串口启动流程, 假设 K3 已经完成 R18 上电观察, root Shell 可用
- 本 runbook 的**所有具体动作、IP 网段、U 盘设备节点 (`/dev/sda1`)、`scp` 示例命令均为典型描述, 未在真板执行**
