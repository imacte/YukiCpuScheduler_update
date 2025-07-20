# YukiCpuScheduler

<div align="center">

[![C++](https://img.shields.io/badge/language-C++-%23f34b7d.svg?style=for-the-badge&logo=cplusplus)](https://en.wikipedia.org/wiki/C++)
[![Android](https://img.shields.io/badge/platform-Android-3DDC84.svg?style=for-the-badge&logo=android)](https://en.wikipedia.org/wiki/Android_(operating_system))
[![AArch64](https://img.shields.io/badge/arch-AArch64-FF6B6B.svg?style=for-the-badge)](https://en.wikipedia.org/wiki/AArch64)
[![Android Support](https://img.shields.io/badge/Android%209~15-Support-4CAF50.svg?style=for-the-badge)](https://developer.android.com/)

**🚀 智能 CPU 调度工具 - 为 Android 设备提供极致的性能与能效平衡**

</div>

---

## 📋 项目简介

YukiCpuScheduler 是一款基于 C++ 编写的智能 CPU 调度工具，专为优化 Android 设备的 CPU 性能和功耗表现而设计。通过先进的调度算法和高度可配置的性能模型，它能够根据不同的使用场景动态调整 CPU 频率、总线速度、核心分配策略以及精细的调速器参数，实现最佳的性能与能效平衡。

## 🔧 系统要求

- **架构支持**: ARM64 平台
- **系统版本**: Android 9.0 - 15.0
- **权限要求**: Root 权限 (Magisk)

## 🎯 情景模式

| 模式 | 图标 | 描述 | 适用场景 |
|------|------|------|----------|
| **Powersave** | 🔋 | 省电模式 | 在保证基本流畅度的同时，尽可能降低功耗 - 推荐**待机**或**轻度**使用 |
| **Balance** | ⚖️ | 均衡模式 | 提供比原厂设置更流畅且更省电的体验 - 推荐**日常**使用 |
| **Performance** | ⚡ | 性能模式 | 提供卓越的流畅度和响应速度，功耗会相应增加 - 推荐**大型应用**或**短时游戏** |
| **Fast** | 🚀 | 极速模式 | 释放全部性能潜力，所有核心以最高频率运行，忽略能效 - 推荐**极限性能测试** |

## 📊 日志等级

| 等级 | 描述 |
|------|------|
| `DEBUG` | 用于开发调试的详细信息，包括 `Govsets` 的写入日志。 |
| `INFO` | 程序运行的关键状态信息。 |
| `WARNING` | 可能存在的问题或异常情况。 |
| `ERROR` | 影响程序运行的错误。 |

---

## ❓ 常见问题解答

<details>
<summary><strong>💡 是否会对待机功耗产生负面影响？</strong></summary>
YukiCpuScheduler 自身采用 C++ 编写，运行功耗极低。通过优化的省电模式，其待机功耗表现通常优于或持平于原厂设置，不会产生负面影响。
</details>

<details>
<summary><strong>🔋 为什么使用了 YukiCpuScheduler 后功耗仍然很高？</strong></summary>
功耗主要由应用负载决定。YukiCpuScheduler 通过智能控制性能释放来优化能效，但无法减少应用本身的计算量。如果后台应用持续高负载运行，功耗自然会很高。
</details>

<details>
<summary><strong>⚙️ 是否还需要关闭系统的 performance boost？</strong></summary>
不需要。YukiCpuScheduler 在初始化阶段会自动禁用大部分主流的用户态和内核态性能增强（如 touch boost），以避免冲突，确保调度策略的唯一性。
</details>

<details>
<summary><strong>🔄 AffinitySetter 功能是否与其他系统流畅度提升模块冲突？</strong></summary>
**会冲突**。`AffinitySetter` 会对一些系统关键进程（如 `surfaceflinger`, `system_server`）进行静态的**进程级**核心绑定优化。为避免冲突，请勿同时使用其他具有类似功能的模块或线程调度模块。
</details>

<details>
<summary><strong>🔄 切换情景模式后是否需要重启？</strong></summary>
**不需要**。你可以通过修改 `/data/adb/modules/YukiCpuScheduler/config.txt` 文件的内容来实时切换模式，模块会自动监听文件变化并应用新设置。
</details>

<details>
<summary><strong>🚀 应用启动加速的频率机制是怎样的？</strong></summary>
应用启动时，会根据您当前模式设定的**最大频率**进行临时超频。默认倍率是 `1.2` 倍，持续 `200` 毫秒。您可以在 `AppLaunchBoostSettings` 中自定义这些值。
</details>

---

## 📁 配置文件详解 (`config.yaml`)

### 1️⃣ 元信息 (`meta`)

```yaml
meta:
  name: "YukiCpuScheduler Profile"
  author: "yuki"
  configVersion: 19
  loglevel: "INFO" 
```
| 字段 | 类型 | 描述 |
|:---|:---|:---|
| `configVersion` | number | **至关重要**。此版本号必须与程序要求的版本完全一致。 |
| `loglevel` | string | 日志记录详细程度。可选值：`DEBUG`, `INFO`, `WARNING`, `ERROR`。 |

### 2️⃣ 功能开关 (`function`)

此部分包含了所有主要功能的总开关。

```yaml
function:
  DisableQcomGpu: true
  AffinitySetter: true
  CpuIdleScaling_Governor: false
  EasScheduler: true
  cpuset: true
  LoadBalancing: true
  EnableFeas: false
  AdjIOScheduler: true
  AppLaunchBoost: true
```
| 功能 | 类型 | 详细描述 |
|:---|:---|:---|
| `DisableQcomGpu` | bool | **(推荐)** 禁用高通 GPU 自有 Boost 机制，避免冲突。 |
| `AffinitySetter` | bool | **(推荐)** 对系统关键进程进行静态核心绑定，**显著提升UI流畅度**。 |
| `CpuIdleScaling_Governor`| bool | 是否允许自定义 CPU Idle 调速器。详见 `CpuIdle` 部分。 |
| `EasScheduler` | bool | 如果内核支持 **EAS**，开启可应用优化参数。详见 `EasSchedulerValue` 部分。 |
| `cpuset` | bool | **(推荐)** 启用 Cpuset 功能，为不同任务组分配合适的 CPU 核心。详见 `Cpuset` 部分。 |
| `LoadBalancing` | bool | 启用 CFS 负载均衡优化，让任务在核心间的分配更合理。 |
| `EnableFeas` | bool | 是否在**极速模式**下尝试启用内核的 FEAS 功能。 |
| `AdjIOScheduler` | bool | 是否允许自定义 I/O 调速器。详见 `IO_Settings` 部分。 |
| `AppLaunchBoost` | bool | **(推荐)** 启用应用启动加速，加快加载速度。 |

### 3️⃣ 核心框架与分配 (`CoreFramework` & `CoreAllocation`)

此部分定义了您设备的物理核心架构，是所有频率和核心控制功能的基础。**必须正确配置！**

#### **核心框架 (`CoreFramework`)**
您需要在这里告诉程序，您设备的不同核心簇分别对应哪个 `policy` 路径。
*   **如何查找?** 在 root 终端中查看 `/sys/devices/system/cpu/cpufreq/` 目录。

```yaml
CoreFramework:
  SmallCorePath: 0
  MediumCorePath: 2
  BigCorePath: 5
  SuperBigCorePath: 7
```

#### **核心分配 (`CoreAllocation`)**
此部分为 `AffinitySetter` 功能提供参数。

```yaml
CoreAllocation:
  cpusetCore: "2-7"
  cpuctlUclampBoostMin: "0"
  cpuctlUclampBoostMax: "100"
```
| 字段 | 类型 | 描述 |
|:---|:---|:---|
| `cpusetCore` | string | `AffinitySetter` 会将系统关键进程绑定到这个指定的核心范围上。 |
| `cpuctlUclampBoost...` | string | Uclamp 相关的提权参数，通常保持默认。 |


### 4️⃣ 总线频率控制 (`Bus_dcvs_Path` & `Bus_dcvs`)

此功能允许您精细控制SoC内部数据总线（LLCC缓存/DDR内存）的频率，对系统响应速度和功耗有显著影响。该功能的配置分为**全局路径定义**和**模式内数值设定**两个步骤。

#### **第一步：全局路径定义 (`Bus_dcvs_Path`)**

此部分用于**一次性**告诉程序，控制总线频率的系统文件位于何处。**程序会智能判断您填写了几个路径，并只对已填写的路径进行操作**。

| 字段名 | 数据类型 | 描述 |
|:---|:---|:---|
| `CPUllccminPath` | string | 指向 **LLCC 缓存最小频率** 控制文件的**路径** |
| `CPUllccmaxPath` | string | 指向 **LLCC 缓存最大频率** 控制文件的**路径** |
| `CPUddrminPath` | string | 指向 **DDR 内存最小频率** 控制文件的**路径** (天玑平台下可指向`set_freq`文件) |
| `CPUddrmaxPath` | string | 指向 **DDR 内存最大频率** 控制文件的**路径** |

**配置示例（高通平台）：**
```yaml
Bus_dcvs_Path:
  CPUllccminPath: "/sys/devices/system/cpu/bus_dcvs/L3/soc:qcom,memlat:l3:prime/min_freq"
  CPUllccmaxPath: "/sys/devices/system/cpu/bus_dcvs/L3/soc:qcom,memlat:l3:prime/max_freq"
  CPUddrminPath: "/sys/devices/system/cpu/bus_dcvs/DDR/soc:qcom,memlat:ddr:prime/min_freq"
  CPUddrmaxPath: "/sys/devices/system/cpu/bus_dcvs/DDR/soc:qcom,memlat:ddr:prime/max_freq"
```
**配置示例（天玑平台）：**
```yaml
Bus_dcvs_Path:
  CPUddrminPath: "/sys/class/devfreq/mtk-dvfsrc-devfreq/userspace/set_freq"
  # 其他留空
```

---

#### **第二步：模式内数值设定 (`Bus_dcvs`)**

在**每一个性能模式**内部，设定在该模式下希望写入的**具体频率数值**。

| 字段名 | 数据类型 | 描述 |
|:---|:---|:---|
| `CPUllccmin` | int | **LLCC 缓存** 的最小频率 (单位: KHz) |
| `CPUllccmax` | int | **LLCC 缓存** 的最大频率 (单位: KHz) |
| `CPUddrmin` | int | **高通**: **DDR** 的最小频率 (KHz)。<br>**天玑**: 写入 `set_freq` 的**目标频率** (Hz)。|
| `CPUddrmax` | int | **DDR 内存** 的最大频率 (单位: KHz) |

**配置示例（高通 `performance` 模式）：**
```yaml
performance:
  Bus_dcvs:
    CPUllccmin: 1555000
    CPUllccmax: 1708800
    CPUddrmin: 1555000
    CPUddrmax: 3196000```
**配置示例（天玑 `performance` 模式）：**
```yaml
performance:
  Bus_dcvs:
    CPUddrmin: 800000000 # 目标频率 (单位: Hz)
```

### 5️⃣ 动态调速器参数 (`pGovPath` & `Govsets`) - 🚀 新核心功能

此功能允许您对 CPU 调速器的内部参数进行精细化调整。它通过**按调速器分组**的两部分协同工作。

#### **第一步：定义可用参数 (`pGovPath`)**
您的“参数字典”，按**调速器名称**分组，定义所有**可能**会用到的参数。

-   **键 (Key)**: 一个抽象的名称，如 `path1`。
-   **值 (Value)**: 内核参数文件相对于其**调速器目录**的【纯文件名】。
-   **模板**: 值为空字符串 `""` 的条目是为用户预留的模板。

**配置示例：**
```yaml
pGovPath:
  schedutil:
    path1: "up_rate_limit_us"
  walt:
    path1: "target_loads"
```

---

#### **第二步：在模式中设置参数值 (`Govsets`)**
在**每一个性能模式**内部，同样按**调速器名称**分组，使用 `pGovPath` 中定义的**键**来设置具体**数值**。

-   程序会**智能地**将 `schedutil` 组的设置，只应用到正在使用 `schedutil` 的核心上。
-   如果值是空字符串 (`""`)，程序会**忽略**此项设置。

**配置示例 (`performance` 模式)：**
```yaml
performance:
  Governor:
    global: "schedutil"
    SuperBigCore: "walt" 

  Govsets:
    schedutil:
      path1: "0"      # up_rate_limit_us
    walt:
      path1: "95"     # target_loads
```

### 6️⃣ Cpuset (核心分组)

此功能需要 `function.cpuset` 为 `true`。它将不同类型的任务组限制在指定的 CPU 核心上运行，是功耗和性能管理的关键。

```yaml
Cpuset:
  top_app: "0-7"
  foreground: "0-7"
  background: "0-2"
  system_background: "1-2"
  restricted: "0-5"```
| 字段 | 描述 | 建议值 |
|:---|:---|:---|
| `top_app` | 当前在前台运行的应用。 | 应分配所有核心，如 `"0-7"`。 |
| `foreground` | 前台服务和可见的应用。 | 也应分配所有或大部分核心。 |
| `background` | 后台运行的应用和服务。 | **应限制在能效核心**，如 `"0-3"` 或 `"0-2"`，以节省功耗。 |
| `system_background` | 系统后台服务。 | 同样应限制在能效核心。 |
| `restricted` | 被系统限制的后台应用。 | 应分配最少的核心。 |

### 7️⃣ EAS 调度器参数 (`EasSchedulerValue`)

此功能需要 `function.EasScheduler` 为 `true`，且您的内核支持 EAS。这些是高级内核参数，通常保持默认即可。

```yaml
EasSchedulerValue:
  sched_min_granularity_ns: "2000000"
  sched_nr_migrate: "30"
  sched_wakeup_granularity_ns: "3200000"
  sched_schedstats: "0"
```
| 字段 | 简要描述 |
|:---|:---|
| `sched_min_granularity_ns` | 任务在被抢占前可运行的最短时间。 |
| `sched_nr_migrate` | 负载均衡时一次性迁移的任务数量。 |
| `sched_wakeup_granularity_ns`| 唤醒的任务在抢占当前任务前需要等待的时间。 |
| `sched_schedstats` | 是否开启调度器统计。设为 `"0"` 可减少开销。 |

### 8️⃣ I/O 设置 (`IO_Settings`)

```yaml
IO_Settings:
  Scheduler: "mq-deadline"
  IO_optimization: true
```
| 字段 | 类型 | 描述 |
|:---|:---|:---|
| `Scheduler` | string | 需要 `function.AdjIOScheduler` 为 `true`。设置块设备的I/O调度器（如 `mq-deadline`, `bfq`, `none`）。留空则使用内核默认。 |
| `IO_optimization` | bool | 开启一组通用的 I/O 优化，如禁用 iostats 等。 |

### 9️⃣ CPU 空闲设置 (`CpuIdle`)

```yaml
CpuIdle:
  current_governor: "teo"
```
| 字段 | 类型 | 描述 |
|:---|:---|:---|
| `current_governor` | string | 需要 `function.CpuIdleScaling_Governor` 为 `true`。设置 CPU Idle 调速器（如 `menu`, `teo`），它决定了CPU在空闲时进入的节能状态深度。留空则使用内核默认。 |

### 🔟 其他设置 (`Other`)

```yaml
Other:
  AdjQcomBus_dcvs: false
```
| 字段 | 类型 | 描述 |
|:---|:---|:---|
| `AdjQcomBus_dcvs`| bool | **(仅高通)** 开启后，会应用一组**硬编码在程序中**的总线（DDR/L3缓存）频率优化，适用于部分高通设备。这是一个遗留的快速优化选项。 |

### 1️⃣1️⃣ 功耗模型详解 (以 `performance` 模式为例)

一个完整的性能模式，是由以下**六个模块**共同定义的。您可以自由组合，打造最适合您的模式。

```yaml
performance:
  Governor: { ... }
  Freq: { ... }
  Uclamp: { ... }
  Bus_dcvs: { ... }
  Govsets: { ... }
  Other: { ... }
```

#### **模块一: 调速器 (`Governor`)**
决定CPU频率如何响应负载。`global`为默认值，`SmallCore`等可覆盖。
```yaml
  Governor:
    global: "schedutil"
    SuperBigCore: "walt" 
```

#### **模块二: CPU频率 (`Freq`)**
定义每个核心簇的最小/最大频率 (单位 KHz，可用 "min" 或 "max")。
```yaml
  Freq:
    SmallCoreMaxFreq: "max"
```

#### **模块三: Uclamp (CPU使用率限制)**
向调度器提供性能需求的提示 (0-100)。
```yaml
  Uclamp:
    UclampTopAppMin: "10"
    UclampTopApplatency_sensitive: "1" 
```

#### **模块四: 总线频率 (`Bus_dcvs`)**
设置此模式下，SoC内部数据总线（LLCC缓存/DDR内存）的频率。更高的值可以显著降低延迟。

```yaml
  Bus_dcvs:
    CPUllccmin: 1555000
    CPUddrmin: 1555000
```

#### **模块五: 调速器参数 (`Govsets`)**
精细化调整当前模式下，每个调速器的具体行为。
```yaml
  Govsets:
    schedutil:
      path1: "0"
    walt:
      path1: "95"
```

#### **模块六: 其他 (`Other`)**
```yaml
  Other:
    ufsClkGate: true # UFS 时钟门控开关
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
- 各位酷安网友以及 YukiCpuScheduler 的所有用户

---

## 📄 开源协议


---

<div align="center">

**🌟 如果这个项目对你有帮助，请给我们一个 Star！**

[![Star History Chart](https://api.star-history.com/svg?repos=Yuki-Chan-233/Yuki-CpuScheduler&type=Date)](https://star-history.com/#Yuki-Chan-233/Yuki-CpuScheduler&Date)

---

<sub>文档更新时间：2025/07/21</sub><br>
<sub>感谢所有用户的测试反馈，这将推进 YukiCpuScheduler 的持续发展 🚀</sub>

</div>
```