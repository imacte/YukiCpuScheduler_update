# YukiCpuScheduler

<div align="center">

[![C++](https://img.shields.io/badge/language-C++-%23f34b7d.svg?style=for-the-badge&logo=cplusplus)](https://en.wikipedia.org/wiki/C++)
[![Android](https://img.shields.io/badge/platform-Android-3DDC84.svg?style=for-the-badge&logo=android)](https://en.wikipedia.org/wiki/Android_(operating_system))
[![AArch64](https://img.shields.io/badge/arch-AArch64-FF6B6B.svg?style=for-the-badge)](https://en.wikipedia.org/wiki/AArch64)
[![Android Support](https://img.shields.io/badge/Android%208~15-Support-4CAF50.svg?style=for-the-badge)](https://developer.android.com/)

**🚀 智能 CPU 调度工具 - 为 Android 设备提供极致的性能与能效平衡**

</div>

---

## 📋 项目简介

YukiCpuScheduler 是一款基于 C++ 编写的智能 CPU 调度工具，专为优化 Android 设备的 CPU 性能和功耗表现而设计。通过先进的智能调度算法，它能够根据不同的使用场景动态调整 CPU 频率，实现最佳的性能与能效平衡。

## 🔧 系统要求

- **架构支持**: ARM64 平台
- **系统版本**: Android 8.0 - 15.0
- **权限要求**: Root 权限

## 🎯 情景模式

| 模式 | 图标 | 描述 | 适用场景 |
|------|------|------|----------|
| **Powersave** | 🔋 | 省电模式 | 在保证基本流畅度的同时，尽可能降低功耗 - 推荐**待机**使用 |
| **Balance** | ⚖️ | 均衡模式 | 比原厂设置更流畅且更省电 - 推荐**日常**使用 |
| **Performance** | ⚡ | 性能模式 | 在保证较高流畅度的同时，可能会增加功耗 - 推荐**短时间高性能**场景 |
| **Fast** | 🚀 | 极速模式 | 全力保证游戏流畅度，忽略能效比，所有核心拉升至最高频率 |

## 📊 日志等级

| 等级 | 描述 |
|------|------|
| `DEBUG` | 调试信息 |
| `INFO` | 一般信息 |
| `WARNING` | 警告信息 |
| `ERROR` | 错误信息 |

---

## ❓ 常见问题解答

<details>
<summary><strong>💡 是否会对待机功耗产生负面影响？</strong></summary>

YukiCpuScheduler 进行了低功耗优化，由于使用了 C++ 语言，自身运行功耗很低，并不会对设备的待机功耗产生显著影响。
</details>

<details>
<summary><strong>🔋 为什么使用了 YukiCpuScheduler 后功耗仍然很高？</strong></summary>

SOC 的 AP 部分功耗主要取决于计算量和使用的频点。YukiCpuScheduler 只能通过控制性能释放和改进频率的方式来降低功耗。如果后台应用的计算量很大，可能无法显著延长续航时间。

**建议**: 
- 使用 Scene 工具箱的进程管理器来定位问题
- 全局测试时间不低于一小时，瞬时功耗不具有参考价值
</details>

<details>
<summary><strong>🎮 如何确保我的设备支持 Feas 特性？</strong></summary>

开启 YukiCpuScheduler 的 Feas 开关并切换到极速模式，YukiCpuScheduler 将会自动识别内核的 Feas 接口。如果设备没有 Feas 功能接口，将会在日志中抛出错误。目前 YukiCpuScheduler 已接入大多数内核的 Feas 接口。
</details>

<details>
<summary><strong>⚙️ 是否还需要关闭系统的 performance boost？</strong></summary>

YukiCpuScheduler 在初始化阶段就已经关闭了大部分主流的用户态和内核态升频，如果有非常规的升频需要用户自己关闭。
</details>

<details>
<summary><strong>🔗 YukiCpuScheduler 和 Scene 工具箱是什么关系？</strong></summary>

这两个软件独立运作，没有互相依赖。YukiCpuScheduler 实现了接口可供 Scene 工具箱调用，例如性能模式切换以及分 APP 性能模式。即使不安装 Scene 工具箱也可以实现性能模式切换。
</details>

<details>
<summary><strong>🗑️ RubbishProcess 指的是什么进程？</strong></summary>

进程列表: `kswapd`, `logd`, `kcompactd`, `magiskd`, `zygiskd`, `init`

为防止这些进程占用过高的 CPU 导致异常耗电，默认将这些进程绑定到 0-2 核心。
</details>

<details>
<summary><strong>🔄 AffintySetter 功能是否与其他系统流畅度提升模块冲突？</strong></summary>

**会冲突**。目前 YukiCpuScheduler 会对一些系统关键进程进行绑核操作，所以不必使用此类模块。
</details>

<details>
<summary><strong>💻 支持哪些内核？</strong></summary>

目前 YukiCpuScheduler 支持大部分内核，包括 4.4 及以上版本的内核。
</details>

<details>
<summary><strong>⚠️ 与特定内核的兼容性？</strong></summary>

目前 YukiCpuScheduler 仅在刷入**潘多拉**内核的设备上测试过，可以使用但可能会导致某些核心的最大频率异常。

**注意**: 调度并未适配这些内核提供的 Feas 接口。
</details>

<details>
<summary><strong>🔄 切换情景模式后是否需要重启？</strong></summary>

**不需要**。目前 YukiCpuScheduler 会监听情景模式的配置变化进行实时切换。
</details>

<details>
<summary><strong>🚀 应用启动加速的频率机制？</strong></summary>

应用启动加速会对不同的核心写入对应最大频率的 1.25 倍：
- 小核: `SmallCoreMaxFreq` × 1.25
- 中核: `MediumCoreMaxFreq` × 1.25  
- 大核: `BigCoreMaxFreq` × 1.25
- 超大核: `SuperBigCoreMaxFreq` × 1.25

**示例**: 冷启动时将小核的频率拉升至 `SmallCoreMaxFreq` 的 1.25 倍，以此类推。
</details>

---

## 📁 配置文件详解

### 1️⃣ 元信息 (meta)

```ini
[meta]
name = "YukiCpuScheduler正式版模型"
author = MoWei
configVersion = 13
loglevel = "INFO"
```

| 字段 | 类型 | 描述 |
|------|------|------|
| `name` | string | 配置文件名称 |
| `author` | string | 配置文件作者 |
| `configVersion` | string | 配置文件版本号 |
| `loglevel` | string | 日志等级 (DEBUG/INFO/WARNING/ERROR) |

### 2️⃣ 功能开关 (function)

```ini
[function]
DisableQcomGpu = false
AffintySetter = true
CpuIdleScaling_Governor = false
EasScheduler = true
cpuset = true
LoadBalancing = true
EnableFeas = false
AdjIOScheduler = true
AppLaunchBoost = true
```

| 功能 | 类型 | 描述 |
|------|------|------|
| `DisableQcomGpu` | bool | 禁用高通 GPU Boost，防止 GPU 频率无序升高 |
| `AffintySetter` | bool | 对系统和传感器关键进程进行绑核操作 |
| `CpuIdleScaling_Governor` | bool | 自定义 CPU Idle 调度器 |
| `EasScheduler` | bool | EAS 调度器参数优化 |
| `cpuset` | bool | CPUSet 功能，调整应用的核心分配 |
| `LoadBalancing` | bool | 通过优化 CFS 调度器参数达到负载均衡 |
| `EnableFeas` | bool | FEAS 功能（仅限极速模式） |
| `AdjIOScheduler` | bool | I/O 调度器调整及优化总开关 |
| `AppLaunchBoost` | bool | APP 启动加速，启动时对 CPU 0-7 核心进行 1.2s 升频 |

### 3️⃣ 核心分配参数 (CoreAllocation)

```ini
[CoreAllocation]
cpusetCore = "4-7"
cpuctlUclampBoostMin = "0"
cpuctlUclampBoostMax = "100"
```

| 字段 | 类型 | 描述 |
|------|------|------|
| `cpusetCore` | string | 指定 CPUSet 核心用于核心分配 |
| `cpuctlUclampBoostMin` | string | CPU 使用率控制最小值 (0-100) |
| `cpuctlUclampBoostMax` | string | CPU 使用率控制最大值 (0-100) |

### 4️⃣ 核心架构参数 (CoreFramework)

```ini
[CoreFramework]
SmallCorePath = 0
MediumCorePath = 4
BigCorePath = 0
SuperBigCorePath = 0
```

| 字段 | 类型 | 描述 |
|------|------|------|
| `SmallCorePath` | int | 小核的 CPU 路径 |
| `MediumCorePath` | int | 中核的 CPU 路径 |
| `BigCorePath` | int | 大核的 CPU 路径 |
| `SuperBigCorePath` | int | 超大核的 CPU 路径 |

### 5️⃣ I/O 设置 (IO_Settings)

```ini
[IO_Settings]
Scheduler = ""
IO_optimization = false
```

| 字段 | 类型 | 描述 |
|------|------|------|
| `Scheduler` | string | I/O 调度器类型 (如 ssg、bfq 等，空值表示不修改) |
| `IO_optimization` | bool | 启用 I/O 优化功能 |

### 6️⃣ QcomBus 参数优化 (Other)

```ini
[Other]
AdjQcomBus_dcvs = false
```

| 字段 | 类型 | 描述 |
|------|------|------|
| `AdjQcomBus_dcvs` | bool | 优化 QCOM 设备的 DDR/LLCC/DDRQOS/L3 参数 (7GEN2+ 设备效果最佳) |

### 7️⃣ EAS 调度器参数 (EasSchedulerVaule)

```ini
[EasSchedulerVaule]
sched_min_granularity_ns = "2000000" 
sched_nr_migrate = "30"
sched_wakeup_granularity_ns = "3200000"
sched_schedstats = "0"
```

| 字段 | 类型 | 描述 |
|------|------|------|
| `sched_min_granularity_ns` | string | EAS 调度器最小调度粒度 (纳秒) |
| `sched_nr_migrate` | string | 控制任务在 CPU 核心间迁移的次数 |
| `sched_wakeup_granularity_ns` | string | EAS 调度器调整任务唤醒时间的粒度 (纳秒) |
| `sched_schedstats` | string | 是否启用调度统计信息收集 (0=禁用) |

### 8️⃣ CPU Idle 调度器 (CpuIdle)

```ini
[CpuIdle]
current_governor = ""
```

| 字段 | 类型 | 描述 |
|------|------|------|
| `current_governor` | string | CPU Idle 调度器模式 (高通推荐: qcom-cpu-lpm，联发科推荐: menu，空值表示不调整) |

### 9️⃣ CPUSet 配置 (Cpuset)

```ini
[Cpuset]
top_app = "0-7"
foreground = "0-7"
restricted = "0-5"
system_background = "1-2"
background = "0-2"
```

| 字段 | 类型 | 描述 |
|------|------|------|
| `top_app` | string | 顶层应用可使用的 CPU 核心范围 |
| `foreground` | string | 前台应用可使用的 CPU 核心范围 |
| `restricted` | string | 前台任务加速时可使用的 CPU 核心范围 |
| `system_background` | string | 系统后台进程可使用的 CPU 核心范围 |
| `background` | string | 后台进程可使用的 CPU 核心范围 |

### 🔟 功耗模型开发 (以 performance 模式为例)

```ini
[performance]
scaling_governor = "schedutil"
UclampTopAppMin = "0"
UclampTopAppMax = "100"
UclampTopApplatency_sensitive = "1"
UclampForeGroundMin = "0"
UclampForeGroundMax = "80"
UclampBackGroundMin = "0"
UclampBackGroundMax = "50"
SmallCoreMaxFreq = 10000
MediumCoreMaxFreq = 2500
BigCoreMaxFreq = 2700
SuperBigCoreMaxFreq = 2700
ufsClkGate = false
```

| 字段 | 类型 | 描述 |
|------|------|------|
| `scaling_governor` | string | 指定 0-7 核心的 CPU 调速器 |
| `UclampTopAppMin/Max` | string | 顶层 APP 可使用的 CPU 频率下限/上限 (0-100) |
| `UclampTopApplatency_sensitive` | string | 延迟敏感性参数，告知调度器前台应用对延迟敏感 |
| `UclampForeGroundMin/Max` | string | 前台 APP 可使用的 CPU 频率下限/上限 (0-100) |
| `UclampBackGroundMin/Max` | string | 后台 APP 可使用的 CPU 频率下限/上限 (0-100) |
| `SmallCoreMaxFreq` | int | 小核 CPU 最大频率 (0-10000) |
| `MediumCoreMaxFreq` | int | 中核 CPU 最大频率 (0-10000) |
| `BigCoreMaxFreq` | int | 大核 CPU 最大频率 (0-10000) |
| `SuperBigCoreMaxFreq` | int | 超大核 CPU 最大频率 (0-10000) |
| `ufsClkGate` | bool | UFS 时钟门设置 |

---

## 🧵 进阶功能：特定线程核心分配

### 📍 配置文件位置
```
/data/adb/modules/YukiCpuScheduler/configs/threads.ini
```

### 🎯 功能说明
除了全局的 CPU 调度策略外，本调度器还支持为特定应用程序的特定线程绑定到指定的 CPU 核心上运行。这项功能对于游戏或专业应用中的关键线程（如渲染线程、UI 主线程）特别有用，可以有效减少线程切换带来的性能抖动。

### 📝 基本语法
- `[应用包名]`: 使用中括号定义配置段
- `线程名 = 核心范围`: 每条规则的格式

### 🏆 三种规则优先级

#### 1. 精确线程绑定 (最高优先级)
```ini
[com.tencent.tmgp.speedmobile]
# 将名字精确为 "UnityMain" 的线程绑定到核心 6-7
UnityMain = 6-7
```

#### 2. 通配符模糊匹配 (中等优先级)
```ini
[com.tencent.tmgp.speedmobile]
# 匹配所有以 "Render" 开头的线程
Render* = 7
# 匹配所有名字中包含 "Audio" 的线程
*Audio* = 0-1
```

#### 3. 应用默认核心组 (最低优先级)
```ini
[com.tencent.tmgp.speedmobile]
# 为应用中所有其他未匹配的线程设置默认核心范围
* = 2-5
```

### 📋 完整配置示例

```ini
[meta]
name = Yuki App Profile Configuration
author = Yuki
version = 2.0

# === QQ飞车配置 ===
[com.tencent.tmgp.speedmobile]
# 默认核心范围 (优先级最低)
* = 2-5
# 渲染线程 (优先级中)
Render* = 7
# 关键线程 (优先级最高)
UnityMain = 6-7
UnityGfxDeviceW = 5-7
```

### 🔍 如何查找应用线程名
```bash
# 方法 1
top -H -p [PID]

# 方法 2
ps -T -p [PID]
```

---

## 🏆 致谢

感谢以下贡献者对本项目的帮助：

<table>
<tr>
<td align="center">
<a href="https://github.com/MoWei-2077"><img src="https://avatars.githubusercontent.com/u/MoWei-2077?v=4" width="100px;" alt=""/><br /><sub><b>MoWei-2077</b></sub></a><br />
</td>
<td align="center">
<a href="https://github.com/Asbzyyds"><img src="https://avatars.githubusercontent.com/u/Asbzyyds?v=4" width="100px;" alt=""/><br /><sub><b>ShenEternal</b></sub></a><br />
</td>
<td align="center">
<a href="https://github.com/zgquangui"><img src="https://avatars.githubusercontent.com/u/zgquangui?v=4" width="100px;" alt=""/><br /><sub><b>AquaPuff🐋</b></sub></a><br />
</td>
<td align="center">
<a href="https://github.com/hfdem"><img src="https://avatars.githubusercontent.com/u/hfdem?v=4" width="100px;" alt=""/><br /><sub><b>hfdem</b></sub></a><br />
</td>
</tr>
<tr>
<td align="center">
<a href="https://github.com/ztc1997"><img src="https://avatars.githubusercontent.com/u/ztc1997?v=4" width="100px;" alt=""/><br /><sub><b>ztc1997</b></sub></a><br />
</td>
<td align="center">
<a href="https://github.com/XShePlus"><img src="https://avatars.githubusercontent.com/u/XShePlus?v=4" width="100px;" alt=""/><br /><sub><b>XShe</b></sub></a><br />
</td>
<td align="center">
<a href="https://github.com/nep-Timeline"><img src="https://avatars.githubusercontent.com/u/nep-Timeline?v=4" width="100px;" alt=""/><br /><sub><b>Timeline</b></sub></a><br />
</td>
<td align="center">
<a href="https://github.com/shrairo"><img src="https://avatars.githubusercontent.com/u/shrairo?v=4" width="100px;" alt=""/><br /><sub><b>shrairo</b></sub></a><br />
</td>
</tr>
</table>

**特别感谢**：
- QQ@长虹久奕
- QQ@Microsoft  
- QQ@:枫
- 各位酷友以及 YukiCpuScheduler 的所有用户

---

## 📄 开源协议

本项目采用开源协议，具体信息请查看 LICENSE 文件。

---

<div align="center">

**🌟 如果这个项目对你有帮助，请给我们一个 Star！**

[![Star History Chart](https://api.star-history.com/svg?repos=YukiCpuScheduler/YukiCpuScheduler&type=Date)](https://star-history.com/#YukiCpuScheduler/YukiCpuScheduler&Date)

---

<sub>文档更新时间：2025/04/18 19:46</sub><br>
<sub>感谢所有用户的测试反馈，这将推进 YukiCpuScheduler 的持续发展 🚀</sub>

</div>