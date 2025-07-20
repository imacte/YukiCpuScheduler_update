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

YukiCpuScheduler 是一款基于 C++ 编写的智能 CPU 调度工具，专为优化 Android 设备的 CPU 性能和功耗表现而设计。通过先进的调度算法和高度可配置的性能模型，它能够根据不同的使用场景动态调整 CPU 频率、总线速度和核心分配策略，实现最佳的性能与能效平衡。

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
| **Fast** | 🚀 | 极速模式 | 释放全部性能潜力，所有核心和总线以最高频率运行，忽略能效 - 推荐**极限性能测试** |

## 📊 日志等级

| 等级 | 描述 |
|------|------|
| `DEBUG` | 用于开发调试的详细信息 |
| `INFO` | 程序运行的关键状态信息 |
| `WARNING` | 可能存在的问题或异常情况 |
| `ERROR` | 影响程序运行的错误 |

---

## ❓ 常见问题解答

<details>
<summary><strong>💡 是否会对待机功耗产生负面影响？</strong></summary>

YukiCpuScheduler 自身采用 C++ 编写，运行功耗极低。通过优化的省电模式，其待机功耗表现通常优于或持平于原厂设置，不会产生负面影响。
</details>

<details>
<summary><strong>🔋 为什么使用了 YukiCpuScheduler 后功耗仍然很高？</strong></summary>

功耗主要由应用负载决定。YukiCpuScheduler 通过智能控制性能释放来优化能效，但无法减少应用本身的计算量。如果后台应用持续高负载运行，功耗自然会很高。

**建议**:
- 使用 `Scene` 等工具箱的进程管理器，检查后台应用活动。
- 进行长时段（至少1小时）的续航测试，瞬时功耗不具参考性。
</details>

<details>
<summary><strong>🎮 如何确保我的设备支持 Feas 特性？</strong></summary>

在 `config.yaml` 中将 `EnableFeas` 设为 `true`，然后切换到极速模式。YukiCpuScheduler 会自动检测内核是否支持 FEAS 接口。如果不支持，将会在日志中记录一条警告信息。
</details>

<details>
<summary><strong>⚙️ 是否还需要关闭系统的 performance boost？</strong></summary>

不需要。YukiCpuScheduler 在初始化阶段会自动禁用大部分主流的用户态和内核态性能增强（如 touch boost），以避免冲突，确保调度策略的唯一性。
</details>

<details>
<summary><strong>🔗 YukiCpuScheduler 和 Scene 工具箱是什么关系？</strong></summary>

两个软件完全独立，没有互相依赖关系。YukiCpuScheduler 提供了接口可供 `Scene` 等第三方工具箱调用，以实现模式切换、分应用调度等功能。您也可以通过直接修改配置文件来切换模式。
</details>

<details>
<summary><strong>🗑️ RubbishProcess 指的是什么进程？</strong></summary>

这是一系列系统核心但易造成异常耗电的进程，如 `kswapd0`, `logd`, `kcompactd0`, `magiskd`, `init` 等。为保证系统稳定性，`AffinitySetter` 功能会将这些进程绑定在能效核心上运行。
</details>

<details>
<summary><strong>🔄 AffintySetter 功能是否与其他系统流畅度提升模块冲突？</strong></summary>

**会冲突**。`AffinitySetter` 会对一些系统关键进程（如 `surfaceflinger`, `system_server`）进行静态绑核优化。为避免冲突，请勿同时使用其他具有类似功能的模块。
</details>

<details>
<summary><strong>💻 支持哪些内核？</strong></summary>

YukiCpuScheduler 设计上兼容大部分主流内核（版本 4.4 及以上），无需特定内核即可运行。
</details>

<details>
<summary><strong>⚠️ 与特定第三方内核的兼容性问题？</strong></summary>

在某些第三方内核（如部分版本的**潘多拉**内核）上使用时，可能会因其自身的调度逻辑导致某些核心的最大频率被异常限制。这并非 YukiCpuScheduler 的问题，而是内核自身的特性。
</details>

<details>
<summary><strong>🔄 切换情景模式后是否需要重启？</strong></summary>

**不需要**。你可以通过修改 `/data/adb/modules/YukiCpuScheduler/config.txt` 文件的内容来实时切换模式，模块会自动监听文件变化并应用新设置。
</details>

<details>
<summary><strong>🚀 应用启动加速的频率机制是怎样的？</strong></summary>

应用启动时，会根据您当前模式设定的**最大频率**进行临时超频。默认倍率是 `1.2` 倍，持续 `200` 毫秒。您可以在 `AppLaunchBoostSettings` 中自定义这些值。
- 小核超频频率 = `SmallCoreMaxFreq` × `FreqMulti`
- 中核超频频率 = `MediumCoreMaxFreq` × `FreqMulti`
- ...以此类推
</s ummary>
</details>

---

## 📁 配置文件详解 (`config.yaml`)

### 1️⃣ 元信息 (`meta`)

此部分定义了配置文件的基本信息。

```yaml
meta:
  name: "Snapdragon 8 Gen 3 Profile"
  author: "yuki"
  configVersion: 18
  loglevel: "INFO" 
```

| 字段 | 类型 | 描述 |
|:---|:---|:---|
| `name` | string | 您的配置文件名称，仅用于标识。 |
| `author` | string | 配置文件作者名。 |
| `configVersion` | number | **至关重要**。此版本号必须与程序要求的版本完全一致，否则程序将拒绝加载。 |
| `loglevel` | string | 日志记录的详细程度。可选值：`DEBUG`, `INFO`, `WARNING`, `ERROR`。 |

### 2️⃣ 功能开关 (`function`)

此部分包含了所有主要功能的总开关，您可以根据需要开启或关闭它们。

```yaml
function:
  DisableQcomGpu: true       
  AffinitySetter: true       
  EasScheduler: true         
  cpuset: true               
  LoadBalancing: true        
  AppLaunchBoost: true       
  EnableThreadAffinity: true
```

| 功能 | 类型 | 详细描述 |
|:---|:---|:---|
| `DisableQcomGpu` | bool | **(推荐开启)** 禁用高通 GPU 自有的 Boost 机制。开启后可避免与本调度冲突，实现更稳定的 GPU 频率控制。 |
| `AffinitySetter` | bool | **(推荐开启)** 对系统关键进程（如`surfaceflinger`, `system_server`）进行静态核心绑定。**显著提升UI流畅度**，减少卡顿。 |
| `CpuIdleScaling_Governor`| bool | 是否允许自定义 CPU Idle 调度器。通常保持默认即可。 |
| `EasScheduler` | bool | 如果您的内核支持 **EAS** (Energy Aware Scheduling)，开启此项可应用一组优化过的调度参数，提升能效。 |
| `cpuset` | bool | **(推荐开启)** 启用 Cgroup 的 Cpuset 功能，为前台、后台等不同任务组分配合适的 CPU 核心。 |
| `LoadBalancing` | bool | 启用 CFS (Completely Fair Scheduler) 负载均衡优化，让任务在核心间的分配更合理。 |
| `EnableFeas` | bool | 是否在**极速模式**下尝试启用内核的 FEAS (Frequency-Aware EAS) 功能，通常用于游戏场景。 |
| `AdjIOScheduler` | bool | 是否允许自定义 I/O 调度器。 |
| `AppLaunchBoost` | bool | **(推荐开启)** 启用应用启动加速。当检测到新应用启动时，临时提高 CPU 频率，加快加载速度。 |
| `EnableThreadAffinity` | bool | **(高级功能)** 是否启用**特定线程核心分配**功能。开启后，程序会读取并应用 `thread.yaml` 文件中的规则。 |

### 3️⃣ 核心框架与分配 (`CoreFramework` & `CoreAllocation`)

此部分定义了您设备的物理核心架构，是所有频率和核心控制功能的基础。**必须正确配置！**

#### **核心框架 (`CoreFramework`)**

您需要在这里告诉程序，您设备的不同核心簇（小核、中核、大核、超大核）分别对应哪个 `policy` 路径。

*   **如何查找?** <br> 在 root 终端中，查看 `/sys/devices/system/cpu/cpufreq/` 目录，其中的 `policyX` 目录就对应一个核心簇。通常 `policy0` 是小核。

```yaml
CoreFramework:
  SmallCorePath: 0      # 小核簇对应的 policy ID
  MediumCorePath: 2     # 中核簇对应的 policy ID
  BigCorePath: 5        # 大核簇对应的 policy ID
  SuperBigCorePath: 7   # 超大核簇对应的 policy ID
```

#### **核心分配 (`CoreAllocation`)**

此部分为 `AffinitySetter` 功能提供参数。

```yaml
CoreAllocation:
  cpusetCore: "2-7" 
```

| 字段 | 类型 | 描述 |
|:---|:---|:---|
| `cpusetCore` | string | 指定一个核心范围（如 `"2-7"`），`AffinitySetter` 会将系统关键进程绑定到这些性能更强的核心上。 |


### 4️⃣ I/O 设置 (`IO_Settings`)

```yaml
IO_Settings:
  Scheduler: "" 
  IO_optimization: true
```

### 5️⃣ 总线频率控制 (`Bus_dcvs_Path` & `Bus_dcvs`) - 🚀 新核心功能

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
# 天玑平台只需填写一个路径，其他留空即可
Bus_dcvs_Path:
  CPUllccminPath: ""
  CPUllccmaxPath: ""
  CPUddrminPath: "/sys/class/devfreq/mtk-dvfsrc-devfreq/userspace/set_freq"
  CPUddrmaxPath: ""
```

---

#### **第二步：模式内数值设定 (`Bus_dcvs`)**

在**每一个性能模式**内部，设定在该模式下希望写入的**具体频率数值**。程序同样只会使用与您已填写路径相对应的值。

| 字段名 | 数据类型 | 描述 |
|:---|:---|:---|
| `CPUllccmin` | int | 用于设置 **LLCC 缓存** 的最小频率 (单位: KHz) |
| `CPUllccmax` | int | 用于设置 **LLCC 缓存** 的最大频率 (单位: KHz) |
| `CPUddrmin` | int | **高通**: 设置 **DDR** 的最小频率 (KHz)。<br>**天玑**: 设置写入 `set_freq` 的**目标频率** (Hz)。|
| `CPUddrmax` | int | 用于设置 **DDR 内存** 的最大频率 (单位: KHz) |

**配置示例（高通 `performance` 模式）：**
```yaml
performance:
  Bus_dcvs:
    CPUllccmin: 1555000
    CPUllccmax: 1708800
    CPUddrmin: 1555000
    CPUddrmax: 3196000
```
**配置示例（天玑 `performance` 模式）：**
```yaml
performance:
  Bus_dcvs:
    # 只需填写与路径对应的 CPUddrmin 值即可
    CPUddrmin: 800000000 # 目标频率 (单位: Hz)
```

### 6️⃣ 功耗模型详解 (以 `performance` 模式为例)

一个完整的性能模式，是由以下几个模块共同定义的。您可以自由组合，打造最适合您的模式。

```yaml
performance:
  # ...
```

#### **模块一: 调速器 (`Governor`)**
决定了CPU频率如何响应负载变化。

*   `global`: 为所有核心簇设置一个默认的调速器。
*   `SmallCore`, `MediumCore`...: 可以为特定核心簇单独设置调速器，其优先级高于`global`。
*   常见值: `schedutil` (能效均衡), `performance` (始终最高频), `powersave` (始终最低频)。

```yaml
  Governor:
    global: "schedutil"
    SuperBigCore: "performance" 
```

#### **模块二: CPU频率 (`Freq`)**
直接定义每个核心簇的最小和最大频率。

*   `...MinFreq`: 设置最低频率。较高的值可以降低响应延迟，但会增加待机功耗。
*   `...MaxFreq`: 设置最高频率。可以限制性能以省电。
*   值可以是具体的频率数值 (单位: KHz)，也可以是 `"min"` 或 `"max"`。

```yaml
  Freq:
    SmallCoreMinFreq: 1804800   
    SmallCoreMaxFreq: "max"      
    # ... 其他核心 ...
```

#### **模块三: Uclamp (CPU使用率限制)**
向调度器提供性能需求的提示，范围是 `0-100`。

*   `...Min`: 设置性能下限。较高的值会强制CPU保持在更高的频率。
*   `...Max`: 设置性能上限。
*   `...TopApp`: 针对前台应用。
*   `...ForeGround`: 针对前台服务。
*   `...BackGround`: 针对后台任务。
*   `UclampTopApplatency_sensitive`: 设为 `"1"` 会告知内核前台应用对延迟敏感，应优先保证响应速度。

```yaml
  Uclamp:
    UclampTopAppMin: "10"
    UclampTopAppMax: "100"
    UclampTopApplatency_sensitive: "1" 
    # ... 其他任务组 ...
```

#### **模块四: 总线频率 (`Bus_dcvs`)**
设置此模式下，SoC内部数据总线（LLCC缓存/DDR内存）的频率。更高的值可以显著降低延迟。

```yaml
  Bus_dcvs:
    CPUllccmin: 1555000
    CPUllccmax: 1708800
    CPUddrmin: 1555000
    CPUddrmax: 3196000
```

#### **模块五: 其他 (`Other`)**

```yaml
  Other:
    ufsClkGate: true
```
*   `ufsClkGate`: UFS 存储时钟门控。设为 `true` 可能在某些场景下提升 I/O 性能，但会略微增加功耗。

---

## 🧵 进阶功能：特定线程核心分配 (`thread.yaml`)

### 📍 配置文件位置
```
/data/adb/modules/YukiCpuScheduler/thread/thread.yaml
```

### 🎯 功能说明
允许您将特定应用的特定线程，强制绑定到指定的 CPU 核心上运行。这对于消除游戏渲染线程、应用 UI 线程的性能抖动至关重要。

**注意**: 此功能需要 `config.yaml` 中的 `EnableThreadAffinity` 设置为 `true` 才会生效。

### 📝 基本语法
YAML 使用`应用包名` -> `核心范围` -> `线程名匹配列表`的层级结构。

```yaml
# meta 信息（可选）
meta:
  name: Yuki App Profile Configuration
  author: Yuki
  version: 2.0

# === 示例: 某游戏 ===
com.tencent.tmgp.somegame:
  # 规则1: 将渲染主线程绑定到性能最强的超大核
  "7":
    - "UnityMain"
    - "RenderThread"
  
  # 规则2: 将其他渲染工作线程和音频线程绑定到性能大核
  "5-6":
    - "UnityGfxDeviceW*" # 使用通配符
    - "*Audio*"
    
  # 规则3 (最低优先级): 将所有其他未被匹配到的线程，默认运行在中核上
  "2-4":
    - "*"
```

### 🔍 如何查找应用线程名
```bash
# 在 adb shell 或 root 终端中执行
top -H -p $(pidof com.some.app.packagename)
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

<sub>文档更新时间：2025/07/20</sub><br>
<sub>感谢所有用户的测试反馈，这将推进 YukiCpuScheduler 的持续发展 🚀</sub>

</div>