# YukiCpuScheduler (Yuki CPU 调度器)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**YukiCpuScheduler 是一个为 Android 设备设计的、高度可配置的后台服务，旨在通过动态调整CPU频率、调速器、Uclamp、Cpuset、CPU亲和性（包括Cgroup级别和新增的线程级别）、I/O调度器和EAS（如果内核支持）参数，来优化设备性能和功耗表现。**

它特别关注于根据当前运行的前台应用（现在可以更精确地使用PID进行识别和优化）和屏幕状态来应用不同的调度策略，以实现在需要高性能时（如游戏、应用启动）提供流畅体验，在日常使用中保持均衡，在设备空闲或息屏时则尽可能节省电量。

---

## ✨ 主要特性 (Features)

* **多模式调度：** 支持预定义多种性能模式（例如 `powersave`, `balance`, `performance`, `fast`），并可根据屏幕状态自动切换。
* **应用特定配置：** 允许用户为特定的应用程序（通过包名识别）定义专属的调度模式或覆盖模式中的个别参数。
* **全面的CPU参数调整：**
    * 支持多核心簇架构 (例如 Small, Medium, Big, SuperBig Cores)。
    * 可配置每个核心簇的 CPU 调速器 (governor)，并支持备选调速器方案。
    * 可配置每个核心簇的最大和最小 CPU 频率。
    * 可配置特定调速器参数 (例如 `schedutil` 的 `up_rate_limit_us`, `down_rate_limit_us` 等，以及内核特有的如 `walt` 和 `uag` 子目录下的参数)。
* **Uclamp 控制：** 根据模式和应用调整 cgroup 的 `cpu.uclamp.min`, `cpu.uclamp.max` 和 `cpu.uclamp.latency_sensitive` (如果内核支持 `cpu.uclamp.latency_sensitive` 或类似的 `cpu.latency_sensitive`)，为调度器提供性能需求提示。
* **Cpuset 控制：** 为不同的进程组 (cgroups 如 top-app, background) 分配特定的 CPU 核心集。支持使用核心类型名称（如 "SmallCore", "BigCore"）进行配置，并能优雅处理设备上不存在的已配置核心类型。
* **CPU 亲和性设置 (双层级)：**
    * **传统Cgroup级别亲和性：** 为特定 cgroup 内的进程设置 CPU 亲和性，将它们绑定到指定的 CPU 核心上。支持使用核心类型名称。
    * **新增线程级别核心分配：** 通过 `ThreadAffinityManager` 实现，允许基于线程名称模式或动态CPU使用率，为特定应用的线程绑定到指定的核心或核心类型。此功能启用时，会优先于传统的Cgroup级别亲和性设置。
* **I/O 调度器调整：**
    * 自动检测块设备。
    * 设置指定的 I/O 调度器 (例如 bfq, kyber)。
    * 可选的 I/O 优化 (例如调整 `nomerges`)。
* **EAS (Energy Aware Scheduling) 参数调整：** 如果内核支持，可以写入自定义的 EAS 相关 sysfs 参数。
* **应用启动加速：** 当检测到新应用启动到前台时，可以临时切换到高性能模式以加速启动过程，之后自动恢复并进入冷却期。
* **配置文件热重载：** 服务在运行时会监视配置文件的更改，并在检测到修改后自动重新加载并应用新的配置，无需重启服务。
* **高度可配置：** 通过主要的 INI 配置文件 (`default_config.ini`, `app_profiles.ini`) 以及可选的处理器特定配置文件 (例如 `sm8650.ini`) 进行详细定制。
* **详细日志：** 支持多种日志级别 (TRACE, DEBUG, INFO, WARN, ERROR, CRITICAL)，方便调试和问题排查。
* **后台守护进程：** 以root权限在后台安静运行。
* **跨平台构建：** 使用 CMake 和 Android NDK 构建，目标为 arm64-v8a (或其他ABI)。
* **服务就绪探测：** 启动时探测关键系统服务（如 PowerManagerService, ActivityManagerService）是否就绪，以增强稳定性。

---

## 📝 配置文件说明

YukiCpuScheduler 主要通过以下 INI 格式的配置文件进行配置，它们通常位于 Magisk 模块的 `/data/adb/modules/YukiCpuScheduler/configs/` 目录 (或者你在 CMakeLists.txt 中指定的模块路径，或服务启动时通过参数指定的路径)：

1.  **`default_config.ini`**: 主配置文件，定义了全局设置、功能开关、默认模式、各模式下的详细参数、cgroup路径、EAS参数、以及新增的线程核心分配全局设置等。
2.  **处理器特定配置文件 (可选)**: 例如 `sm8650.ini`, `sm8450.ini`。如果存在与当前设备处理器型号匹配的此类文件（文件名需与服务识别到的处理器型号字符串完全一致，包括大小写和符号，通常是小写形式，如 `sm8650.ini`），其配置将**覆盖** `default_config.ini` 中的同名设置。服务启动时会尝试加载。
3.  **`app_profiles.ini`**: 应用特定配置文件，用于为指定的应用程序覆盖默认模式或模式中的参数，包括新增的线程级别核心分配规则。此文件中的设置优先级高于处理器特定配置和 `default_config.ini` 中关于应用行为的设置。

详细的配置项说明和示例请参阅下方的配置指南章节。

---

## ⚙️ 如何手动配置 `default_config.ini`

本节详细指导您如何修改 `default_config.ini` 文件（以及可选的处理器特定 `.ini` 文件）。当存在与当前设备处理器型号匹配的特定配置文件（例如 `sm8650.ini`，位于 `configs/` 目录下）时，该特定文件中的设置会**覆盖** `default_config.ini` 中的同名设置。

**重要提示:**

* **备份**: 在进行任何修改之前，强烈建议您备份原始的配置文件。
* **语法**:
    * 配置文件采用INI格式，由多个**节 (Section)** 组成，节名用方括号 `[]` 包围，例如 `[meta]`。
    * 每个节下面包含多个**键值对 (Key-Value Pairs)**，格式为 `键 = 值`，例如 `loglevel = DEBUG`。
    * **注释**: 以分号 `;` 或井号 `#` 开头的行为注释行，将被程序忽略。
    * **值格式**:
        * 除非特别说明，配置值**不应**包含英文双引号 `""` 或单引号 `''`。程序内部会处理首尾空白和引号。
        * 布尔值通常使用 `true` 或 `false` (不区分大小写)。
        * 频率单位通常为 `kHz` (千赫兹)，例如 `1800000` 代表 1.8GHz。
        * 时间单位通常在键名中指明，例如 `Ms` (毫秒) 或 `Sec` (秒)。
* **热重载**: 修改 `default_config.ini` 后，服务会自动检测更改并热重载配置。如果修改了处理器特定配置文件，建议重启服务以确保更改完全生效。

---

### `default_config.ini` 配置文件结构和主要配置项

#### 1. `[meta]` - 元数据和全局设置

* **`name`**, **`author`**, **`configVersion`**: 描述性信息和配置版本。
* **`ModuleRootPath`**: 【**非常重要**】模块在设备上的实际安装根目录。如果留空，程序会尝试推断，但建议明确设置（例如 `/data/adb/modules/YukiCpuScheduler`）。
* **`loglevel`**: 日志级别 (`TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`, `CRITICAL`, `OFF`)。
* **`screen_on_mode`**: 屏幕点亮时应用的默认调度模式名称 (对应后续定义的模式节名)。
* **`screen_off_mode`**: 屏幕关闭时应用的默认调度模式名称。
* **`EnablePerAppProfile`**: 是否启用 `app_profiles.ini` 文件 (`true` 或 `false`)。

#### 2. `[monitor]` - 监控相关设置

* **`ScreenStateMethod`**: 检测屏幕状态的方法 (`DUMPSYS_POWER` 或 `SYSFS`)。
* **`ScreenStateSysfsPath`**: 若 `ScreenStateMethod = SYSFS`，则指定sysfs节点路径。
* **`ScreenCheckIntervalMs`**: 屏幕状态检查间隔（毫秒）。
* **`AppCheckIntervalMs`**: 前台应用切换检查间隔（毫秒）。
* **`MainLoopSleepSec`**: 服务主循环休眠时间（秒）。
* **`EnterStandbyDelayMs`**: 息屏后，延迟多久服务进入定义的“待机模式”（暂停AppWatcher等）。

#### 3. `[monitor_tuning]` - 监控调优参数

本节用于微调服务在启动初期探测依赖服务以及常规监控时执行 `dumpsys` 命令的参数。
* **`InitialProbeMaxRetries`**: 服务启动时，对依赖系统服务进行就绪状态探测的最大重试次数。
* **`InitialProbeDelayMs`**: 每次初始服务就绪探测尝试之间的延迟（毫秒）。
* **`DumpsysPowerProbeCommand`**: 用于探测 `PowerManagerService` 是否就绪的 `dumpsys` 命令。
* **`DumpsysPowerMonitorCommand`**: 用于常规监控屏幕状态的 `dumpsys power` 命令。
* **`DumpsysPowerMonitorRetries`**: 常规监控时 `dumpsys power` 命令失败后的重试次数。
* **`DumpsysPowerMonitorDelayMs`**: 常规监控时 `dumpsys power` 命令失败后的重试延迟（毫秒）。
* **`DumpsysActivityProbeCommand`**: 用于探测 `ActivityManagerService` 是否就绪的 `dumpsys` 命令。
* **`DumpsysActivityMonitorCommand`**: 用于常规监控前台应用的 `dumpsys activity` 命令 (例如 `dumpsys activity activities` 或 `dumpsys activity top -c`)。
* **`DumpsysActivityMonitorRetries`**: 常规监控时 `dumpsys activity` 命令失败后的重试次数。
* **`DumpsysActivityMonitorDelayMs`**: 常规监控时 `dumpsys activity` 命令失败后的重试延迟（毫秒）。

#### 4. `[function]` - 主要功能开关

一系列布尔值 (`true` 或 `false`)，用于启用或禁用各项核心优化功能。
* **`CpuFreqGovControl`**: CPU频率和调速器控制。
* **`UclampControl`**: Cgroup Uclamp值控制。
* **`CpusetControl`**: Cgroup Cpuset核心绑定控制。
* **`AffinitySetter`**: 传统Cgroup级别CPU亲和性设置。**注意**：如果下面的 `ThreadAffinityControl` 被启用并处于活动模式，此项将被忽略。
* **`AdjIOScheduler`**: I/O调度器调整。
* **`IO_optimization`**: 其他I/O优化。
* **`EasScheduler`**: EAS内核参数设置。
* **`AppLaunchBoost`**: 应用启动加速功能。

#### 5. `[CoreFramework]` - CPU核心框架路径配置

定义不同CPU核心簇的`cpufreq`驱动代表性路径，影响核心类型自动检测与映射。
* **`SmallCorePath`**, **`MediumCorePath`**, **`BigCorePath`**, **`SuperBigCorePath`**:
    * 值可以是 `auto` (推荐，程序自动基于`cpuinfo_max_freq`检测), CPU索引号 (如 `0`), 完整的cpufreq策略路径 (如 `/sys/devices/system/cpu/cpu0/cpufreq`), 或 `-1` (禁用此核心类型的路径配置，不推荐除非特殊情况)。
    * `auto` 模式下，程序会自动识别核心簇。如果手动指定路径无效，该核心类型将无代表性调频路径。

#### 6. `[IO_Settings]` - I/O调度相关设置

* **`BlockDevice`**: 目标块设备名称 (`auto` 自动检测，或手动指定如 `sda`, `mmcblk0`)。
* **`Scheduler`**: 期望设置的I/O调度器名称 (例如 `bfq`, `kyber`, `none` 等)。
* **`ReadAheadKb`**: (可选) 块设备的预读大小（KB）。

#### 7. `[EasSettings]` 和 `[EasSchedulerValue]` - EAS参数设置

调整内核Energy Aware Scheduling (EAS) 相关参数。
* **`[EasSettings]` `BasePaths`**: 逗号分隔的EAS参数基础sysfs路径列表。
* **`[EasSchedulerValue]`**: 定义要设置的EAS参数及其值 (例如 `sched_energy_aware = 1`)。参数名会与 `BasePaths` 中的路径组合尝试写入。

#### 8. `[CgroupPaths]` - Cgroup路径定义

定义各类cgroup的文件系统路径或路径前缀。
* **`XxxCpuCtlPathPrefix`**: Uclamp设置的cgroup路径前缀 (例如 `TopAppCpuCtlPathPrefix = "/dev/cpuctl/top-app"`)。在其后会追加 `/cpu.uclamp.min` 等。
* **`XxxCpusetPath`**: Cpuset设置的cgroup路径 (例如 `BackgroundCpusetPath = "/dev/cpuset/background"`)。在其后会追加 `/cpus` 或 `/tasks`。
    * 支持的 `Xxx` 前缀有 `TopApp`, `Foreground`, `Background`, `SystemBg`, `Restricted`。

#### 9. `[CpusetSettings]` - 默认Cpuset核心绑定

为cgroup类别设置默认的CPU核心绑定。这些设置可以被特定模式中的 `Cpuset_CgroupCategoryKey`覆盖。
* 键: cgroup类别名 (例如 `TopApp`, `Background`，对应`[CgroupPaths]`中的前缀)。
* 值: CPU核心列表字符串。**支持核心类型名称** (如 "SmallCore", "BigCore", "SuperBigCore"), 特殊关键字 (如 "SmallestSingle", "AllCores"), 以及传统数字和范围 (如 "0-3", "7")。这些符号名由 `CpuUtils::CpuListParser` 解析。
    * 程序会根据 `[CoreFramework]` 的检测结果动态解析核心类型名称。若配置中指定的核心类型名称在当前设备未检测到，该名称将被跳过。
    * 示例: `TopApp = "MediumCore,BigCore,SuperBigCore"`

#### 10. `[AffinitySettings]` - CPU亲和性设置 (传统Cgroup级别)

为cgroup类别（或结合模式）设置CPU亲和性。**注意**：如果 `[ThreadAffinityControl] Enable = true` 且 `Mode` 不是 `off`，则此节的设置将被忽略。
* 格式与值的解析逻辑同 `[CpusetSettings]` (同样支持核心类型名称和跳过逻辑)。
    * 默认键: `Affinity_CgroupCategoryKey = "核心列表字符串"` (例如 `Affinity_Background = "SmallCore"`)。
    * 模式特定键: `Affinity_CgroupCategoryKey_ModeName = "核心列表字符串"` (例如 `Affinity_TopApp_performance = "BigCore"`)。

#### 11. `[AppLaunchBoostSettings]` - 应用启动加速设置

控制应用启动加速功能。
* **`Enable`**: 是否启用 (`true`/`false`)。
* **`BoostDurationMs`**: 加速持续时间（毫秒）。
* **`BoostModeName`**: 加速时应用的调度模式名称 (必须是本配置文件中已定义的模式)。
* **`CoolDownMs`**: 一次加速结束后，到下一次可以触发加速的冷却时间（毫秒）。
* **`ExcludedApps`**: 逗号分隔的不触发启动加速的应用包名列表。

#### 12. `[ThreadAffinityControl]` - 线程级别核心分配控制

全局控制新的线程级别核心分配功能。
* **`Enable`**: (`true`/`false`) 总开关，启用或禁用 `ThreadAffinityManager`。
* **`Mode`**: 核心分配模式，可选值：
    * `off`: 完全禁用自定义线程核心分配。
    * `manual`: 完全根据用户在 `app_profiles.ini` 中为特定应用定义的 `ThreadRule_` 规则分配核心。
    * `automatic`: 完全根据线程的实时CPU占用率动态分配核心，使用 `[DynamicThreadAffinity]` 节中的参数。
    * `hybrid`: 混合模式。优先应用 `app_profiles.ini` 中的 `ThreadRule_` 手动规则；对于应用内未匹配任何手动规则的线程，则尝试应用 `[DynamicThreadAffinity]` 的自动动态分配逻辑。
* **`TargetProcessName`**: (可选) 字符串。如果设置了进程名或包名 (例如 `com.example.game` 或 `surfaceflinger`)，则 `ThreadAffinityManager` 的自动和手动（如果规则与此名匹配）模式将主要关注此指定进程的线程。如果留空，则：
    * `automatic` 和 `hybrid` 模式下的自动分配部分，通常会监控当前前台应用的线程。
    * `manual` 和 `hybrid` 模式下的手动规则部分，仍然依赖于 `app_profiles.ini` 中每个节（包名）下的 `ThreadRule_`。

#### 13. `[DynamicThreadAffinity]` - 自动动态线程亲和性设置

当 `[ThreadAffinityControl] Mode` 为 `automatic` 或 `hybrid` 时，本节参数用于控制线程的动态核心分配。
* **`ScanIntervalMs`**: 线程CPU占用率的扫描和分配决策的时间间隔（毫秒）。
* **`HighUsageThreshold`**, **`MediumUsageThreshold`**: CPU占用率百分比阈值 (0-100)，用于将线程分类为高、中、低占用。
* **`HighUsageTargetCores`**, **`MediumUsageTargetCores`**, **`LowUsageTargetCores`**: 为不同占用率类别的线程指定的符号化目标核心列表（例如 `"SuperBigCore,BigCore"`, `"MediumCore"`, `"SmallCore"`）。由 `CpuUtils::CpuListParser` 解析。
* **`StickHighUsageToSingleCore`**, **`StickMediumUsageToSingleCore`**: (`true`/`false`) 是否将高/中占用线程严格绑定到其目标核心组中的**单个**核心。如果为 `false`，则线程亲和性会设为目标核心组的全部核心。
* **`HighUsageCoreSelectionPolicy`**, **`MediumUsageCoreSelectionPolicy`**: 如果 `Stick...ToSingleCore` 为 `true` 且目标核心组有多个核心，此策略决定如何选择那一个核心。可选值: `"first"` (选择列表中的第一个), `"round_robin"` (基于线程TID哈希的轮询选择), `"least_loaded"` (高级，选择负载最低的，可能尚未完全实现或依赖特定内核支持)。
* **`CoolDownMsAfterMigration`**: 线程被自动迁移核心后，在多少毫秒内不再对其进行下一次自动迁移，以避免过于频繁的切换。

#### 14. `[ManualThreadAffinity]` - 手动静态线程亲和性设置

当 `[ThreadAffinityControl] Mode` 为 `manual` 或 `hybrid` 时，本节参数提供全局默认值。
* **`DefaultAffinityForUnmatchedGlobal`**: 符号化核心列表字符串。用于那些在 `app_profiles.ini` 中没有特定 `ThreadRule_` (包括 `ThreadRule_Default`) 覆盖的线程。
    * 在 `manual` 模式下，如果一个应用没有自己的 `ThreadRule_Default`，其未匹配线程会使用此全局默认。
    * 在 `hybrid` 模式下，如果一个应用没有自己的 `ThreadRule_Default`，其未匹配线程会先尝试此全局默认；如果此全局默认也是空或未定义（或解析为允许所有核心），则这些线程可能会进入自动动态分配逻辑。
    * 例如: `DefaultAffinityForUnmatchedGlobal = "AllCores"`

#### 15. `[UclampSettings_Default]` - 全局Uclamp默认值

为不同cgroup类别定义Uclamp参数的默认值。如果某个模式（如下面的模式定义）中没有为特定cgroup类别指定Uclamp参数，则会使用此处的默认值。
* 键格式: `UclampMin_CgroupCategoryKey`, `UclampMax_CgroupCategoryKey`, `UclampLatencySensitive_CgroupCategoryKey`。
* `CgroupCategoryKey` 可以是 `TopApp`, `Foreground`, `Background`, `SystemBg`, `Restricted`。
* 示例: `UclampMin_TopApp = 0`, `UclampMax_Background = 50`

#### 16. 模式定义 (例如 `[powersave]`, `[balance]`, `[performance]`, `[fast]`)

定义不同场景下的系统行为。每个模式都是一个独立的节。
* 在一个模式节内:
    * **调速器设置**:
        * `ScalingGovernor = governor_name` (例如 `schedutil`, `performance`, `powersave`, `walt`, `uag`)：此模式下所有核心簇的默认主调速器。
        * `FallbackScalingGovernor = governor_name`: 如果主调速器设置失败，则尝试此备选调速器。
        * `CoreTypeScalingGovernor = governor_name`: 可为特定核心类型 (如 `SmallCoreScalingGovernor`) 指定主调速器，覆盖上面的全局模式默认。
        * `CoreTypeFallbackScalingGovernor = governor_name`: 特定核心类型的备选调速器。
    * **频率设置**:
        * `CoreTypeMaxFreq = frequency_in_kHz | max`: 特定核心类型的最大频率。`max` 表示使用内核允许的最高频率。
        * `CoreTypeMinFreq = frequency_in_kHz | min`: 特定核心类型的最小频率。`min` 表示使用内核允许的最低频率。
    * **特定调速器参数 (重要！)**：
        * 这些参数仅当对应核心簇**最终生效**的调速器是其目标类型时才会被应用。
        * **Schedutil 特定参数**: 键名格式 `CoreTypeSchedutilParameterNameInCamelCase` (例如 `SmallCoreSchedutilUpRateLimitUs`, `BigCoreSchedutilHispeedFreq`)。程序会将 `ParameterNameInCamelCase` 转换为下划线形式的节点名（如 `up_rate_limit_us`）并尝试写入到 `/sys/.../cpufreq/policyX/schedutil/` 目录下。
        * **WALT 特定参数**: 键名格式 `CoreTypeWaltParameterNameInCamelCase` (例如 `MediumCoreWaltTargetLoads`)。写入到 `/sys/.../cpufreq/policyX/walt/` 目录。
        * **UAG 特定参数**: 键名格式 `CoreTypeUagParameterNameInCamelCase` (例如 `BigCoreUagStallAware`)。写入到 `/sys/.../cpufreq/policyX/uag/` 目录。
    * **模式特定的Uclamp设置**: 键名格式 `UclampMin_CgroupCategoryKey`, `UclampMax_CgroupCategoryKey`, `UclampLatencySensitive_CgroupCategoryKey` (例如 `UclampMin_TopApp = 10`)。这些会覆盖 `[UclampSettings_Default]` 中的值。
    * **模式特定的Cpuset/Affinity设置 (可选)**:
        * `Cpuset_CgroupCategoryKey = "核心列表字符串"` (覆盖 `[CpusetSettings]` 中的同名项)。
        * `Affinity_CgroupCategoryKey = "核心列表字符串"` (覆盖 `[AffinitySettings]` 中的同名项，但同样受 `ThreadAffinityControl` 的影响)。
    * **直接写入Sysfs节点**: `SysfsPath_/path/to/sysfs/node = value_to_write` (路径必须是绝对路径)。这些写入操作通常在其他参数设置之后执行。

---

## ⚙️ 如何手动配置 `app_profiles.ini`

本文件用于定义特定应用程序在前台运行时的专属调度行为，它可以覆盖 `default_config.ini`（及处理器特定配置）中的设定。

**重要提示:**

* 确保 `default_config.ini` 中的 `[meta] EnablePerAppProfile = true`。
* 配置优先级：`app_profiles.ini` 中针对某应用的设置 > 模式中针对某应用的参数（如果有这种细分，目前主要通过直接覆盖模式参数实现） > 处理器特定配置中的模式参数 > `default_config.ini` 中的模式参数。

---

### `app_profiles.ini` 配置文件结构和主要配置项

文件由多个节组成，每个节的名称是目标应用的**包名**。

**在一个应用节内 (例如 `[com.tencent.tmgp.sgame]`)**:

1.  **强制模式切换**:
    * `mode = mode_name`
    * 指定此应用在前台时，强制使用名为 `mode_name` 的调度模式（该模式必须在 `default_config.ini` 或处理器特定配置中已定义）。这将覆盖 `default_config.ini` 中 `[meta] screen_on_mode` 定义的模式。
    * 示例: `mode = performance`

2.  **覆盖当前生效模式的参数**:
    * 您可以列出任何在 `default_config.ini` 模式定义中有效的参数键名，并为其赋予新值。
    * **如果上面指定了 `mode = ...`**：那么这里列出的参数会覆盖那个被强制指定的模式中的对应参数，但仅对此应用生效。
    * **如果没有指定 `mode = ...`**：那么此应用会使用全局的 `screen_on_mode`。这里列出的参数会覆盖那个全局 `screen_on_mode` 中的对应参数，但仅对此应用生效。
    * 参数键名示例 (与 `default_config.ini` 模式定义中的键名一致)：
        * `SmallCoreMaxFreq = 2000000`
        * `BigCoreMinFreq = 1500000`
        * `MediumCoreSchedutilHispeedLoad = 80`
        * `UclampMin_TopApp = 50` (注意：这会影响运行此应用时其所属的TopApp Cgroup的Uclamp设置)
        * `Cpuset_TopApp = "4-7"` (同样影响整个TopApp Cgroup，需谨慎)
        * `Affinity_TopApp = "6-7"` (传统Cgroup级别亲和性，如果未被 `ThreadAffinityControl` 禁用)
        * `SysfsPath_/sys/devices/system/cpu/cpu0/cpufreq/schedutil/hispeed_freq = 1200000` (直接写入sysfs节点)

3.  **线程级别核心分配规则 (Thread Rules)**:
    * 这些规则仅在 `default_config.ini` 中 `[ThreadAffinityControl] Enable = true` 且 `Mode` 为 `manual` 或 `hybrid` 时，由此应用的线程触发。
    * 键名格式: `ThreadRule_线程名模式 = "核心列表字符串"`
    * **`线程名模式`**:
        * 精确名称: 例如 `UnityMain`，则完全匹配名为 "UnityMain" 的线程。
        * 前缀匹配: 例如 `RenderThread*`，则匹配所有以 "RenderThread" 开头的线程名。
    * **`核心列表字符串`**: 与 `[CpusetSettings]` 或 `[AffinitySettings]` 中的值格式相同，支持核心类型名称 ("SmallCore", "BigCore0", "SuperBigCore" 等，其中数字表示该类型核心中的第几个，例如 `BigCore0` 是第一个大核)、CPU ID ("0", "7")、范围 ("0-3")，或它们的组合，由逗号分隔。这些由 `CpuUtils::CpuListParser` 解析。如果设备上没有配置的核心类型，或指定的核心索引超出范围，该部分会被跳过或解析失败。
        * 示例:
            * `ThreadRule_UnityMain = "SuperBigCore0"`
            * `ThreadRule_UnityGfxDeviceW = "BigCore"`
            * `ThreadRule_Job.worker* = "MediumCore"`
            * `ThreadRule_RenderThread* = "BigCore0,BigCore1"`
            * `ThreadRule_binder:* = "0-1"`
    * **应用内默认规则**:
        * `ThreadRule_Default = "核心列表字符串"`
        * 用于此应用内**其他未匹配**到上述任何特定 `ThreadRule_线程名模式` 规则的线程。
        * 如果此应用没有提供 `ThreadRule_Default`，则会查找 `default_config.ini` 中 `[ManualThreadAffinity] DefaultAffinityForUnmatchedGlobal` 的设置作为备选（尤其是在 `manual` 模式下）。在 `hybrid` 模式下，如果两处都没有为线程提供手动规则，则线程会进入自动动态分配逻辑。
        * 示例: `ThreadRule_Default = "MediumCore,SmallCore"`

**配置示例 (`app_profiles.ini`)**

```ini
; --- 王者荣耀示例 ---
[com.tencent.tmgp.sgame]
mode = performance ; 此游戏运行时使用 performance 模式

; 覆盖 performance 模式中的参数 (仅对此游戏生效)
BigCoreMinFreq = 2000000
SuperBigCoreSchedutilUpRateLimitUs = 4000

; 线程分配规则
ThreadRule_UnityMain = "SuperBigCore0"
ThreadRule_UnityGfxDeviceW = "BigCore"
ThreadRule_RenderThread* = "BigCore0,BigCore1"
ThreadRule_bqgThread-* = "SmallCore" ; 假设这是某种后台队列线程
ThreadRule_Default = "MediumCore" ; 此游戏内其他未匹配线程使用所有中核

; --- 某个阅读应用 ---
[com.example.reader]
mode = powersave
BigCoreMaxFreq = 800000 ; 进一步限制大核频率

; 此阅读应用不需要精细的线程分配，所以没有 ThreadRule_
; 如果全局 ThreadAffinityControl Mode 是 manual，且 DefaultAffinityForUnmatchedGlobal 有效，则应用全局默认。
; 如果是 hybrid，则其线程会尝试全局默认，然后可能进入自动分配。
; 如果是 automatic，则其线程完全由自动分配逻辑处理。




---

## 🤝 贡献 (Contributing)

欢迎通过 Pull Requests 贡献代码，或在 Issues 中报告问题和提出建议。

## 📜 许可证 (License)

本项目采用 MIT 许可证。
