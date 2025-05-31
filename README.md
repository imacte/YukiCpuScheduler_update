# YukiCpuScheduler (Yuki CPU 调度器)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**YukiCpuScheduler 是一个为 Android 设备设计的、高度可配置的后台服务，旨在通过动态调整CPU频率、调速器、Uclamp、Cpuset、CPU亲和性、I/O调度器和EAS（如果内核支持）参数，来优化设备性能和功耗表现。** [cite: 1]

它特别关注于根据当前运行的前台应用和屏幕状态来应用不同的调度策略，以实现在需要高性能时（如游戏、应用启动）提供流畅体验，在日常使用中保持均衡，在设备空闲或息屏时则尽可能节省电量。 [cite: 1]

---

## ✨ 主要特性 (Features)

* **多模式调度：** 支持预定义多种性能模式（例如 `powersave`, `balance`, `performance`, `fast`），并可根据屏幕状态自动切换。 [cite: 1]
* **应用特定配置：** 允许用户为特定的应用程序（通过包名识别）定义专属的调度模式或覆盖模式中的个别参数。 [cite: 1]
* **全面的CPU参数调整：**
    * 支持多核心簇架构 (例如 Small, Medium, Big, SuperBig Cores)。 [cite: 1]
    * 可配置每个核心簇的 CPU 调速器 (governor)，并支持备选调速器方案。 [cite: 1]
    * 可配置每个核心簇的最大和最小 CPU 频率。 [cite: 1]
    * 可配置特定调速器参数 (例如 `schedutil` 的 `up_rate_limit_us`, `down_rate_limit_us` 等，以及内核特有的如 `walt` 和 `uag` 子目录下的参数)。 [cite: 1]
* **Uclamp 控制：** 根据模式和应用调整 cgroup 的 `cpu.uclamp.min`, `cpu.uclamp.max` 和 `cpu.uclamp.latency_sensitive` (如果内核支持 `cpu.uclamp.latency_sensitive` 或类似的 `cpu.latency_sensitive`)，为调度器提供性能需求提示。 [cite: 1]
* **Cpuset 控制：** 为不同的进程组 (cgroups 如 top-app, background) 分配特定的 CPU 核心集。支持使用核心类型名称（如 "SmallCore", "BigCore"）进行配置，并能优雅处理设备上不存在的已配置核心类型。 [cite: 1]
* **CPU 亲和性设置：** 为特定 cgroup 内的进程设置 CPU 亲和性，将它们绑定到指定的 CPU 核心上。支持使用核心类型名称，并能优雅处理设备上不存在的已配置核心类型。 [cite: 1]
* **I/O 调度器调整：**
    * 自动检测块设备。 [cite: 1]
    * 设置指定的 I/O 调度器 (例如 bfq, kyber)。 [cite: 1]
    * 可选的 I/O 优化 (例如调整 `nomerges`)。 [cite: 1]
* **EAS (Energy Aware Scheduling) 参数调整：** 如果内核支持，可以写入自定义的 EAS 相关 sysfs 参数。 [cite: 1]
* **应用启动加速：** 当检测到新应用启动到前台时，可以临时切换到高性能模式以加速启动过程，之后自动恢复并进入冷却期。 [cite: 1]
* **配置文件热重载：** 服务在运行时会监视配置文件的更改，并在检测到修改后自动重新加载并应用新的配置，无需重启服务。 [cite: 1]
* **高度可配置：** 通过主要的 INI 配置文件 (`default_config.ini`, `app_profiles.ini`) 以及可选的处理器特定配置文件 (例如 `sdm8gen1.ini`) 进行详细定制。 [cite: 1]
* **详细日志：** 支持多种日志级别 (TRACE, DEBUG, INFO, WARN, ERROR, CRITICAL)，方便调试和问题排查。 [cite: 1]
* **后台守护进程：** 以root权限在后台安静运行。 [cite: 1]
* **跨平台构建：** 使用 CMake 和 Android NDK 构建，目标为 arm64-v8a (或其他ABI)。 [cite: 1]

---

## 📝 配置文件说明

YukiCpuScheduler 主要通过以下 INI 格式的配置文件进行配置，它们通常位于 Magisk 模块的 `/data/adb/modules/YukiCpuScheduler/configs/` 目录 (或者你在 CMakeLists.txt 中指定的模块路径，或服务启动时通过参数指定的路径)： [cite: 1]

1.  **`default_config.ini`**: 主配置文件，定义了全局设置、功能开关、默认模式、各模式下的详细参数、cgroup路径、EAS参数等。 [cite: 1]
2.  **`app_profiles.ini`**: 应用特定配置文件，用于为指定的应用程序覆盖默认模式或模式中的参数。 [cite: 1]
3.  **处理器特定配置文件 (可选)**: 例如 `sdm8gen1.ini`。如果存在与当前设备处理器型号匹配的此类文件，其配置将覆盖 `default_config.ini` 中的同名设置。 [cite: 1]

详细的配置项说明和示例请参考这些文件内的注释，或参阅下方的“如何手动配置 default_config.ini”章节。

## 🛠️ 项目结构

以下是项目的主要目录和文件结构：

* `YukiCpuScheduler/` (项目根目录)
    * `CMakeLists.txt` - CMake 构建脚本 [cite: 1]
    * `README.md` - 本文档 [cite: 1]
    * `main.cpp` - 主程序入口 [cite: 1]
    * `core/` - 核心调度逻辑模块 [cite: 1]
        * `Scheduler.cpp`, `Scheduler.h` - CPU调度策略实现，核心类型检测与映射 [cite: 1]
        * `CpuCoreManager.cpp`, `CpuCoreManager.h` - CPU核心底层控制 (绑核、频率、Uclamp、Cpuset等) [cite: 1]
        * `CgroupManager.cpp`, `CgroupManager.h` - Cgroup (Uclamp, Cpuset) 管理 [cite: 1]
        * `AffinityManager.cpp`, `AffinityManager.h` - CPU亲和性管理 [cite: 1]
        * `AppLaunchManager.cpp`, `AppLaunchManager.h` - 应用启动加速 [cite: 1]
        * `IoManager.cpp`, `IoManager.h` - I/O调度器管理 [cite: 1]
        * `EasManager.cpp`, `EasManager.h` - EAS参数管理 [cite: 1]
    * `config/` - 配置解析模块 [cite: 1]
        * `ConfigParser.cpp`, `ConfigParser.h` - INI配置文件解析器（支持合并特定配置） [cite: 1]
    * `monitor/` - 系统与应用监控模块 [cite: 1]
        * `SystemEvent.cpp`, `SystemEvent.h` - 屏幕状态、待机状态监控 [cite: 1]
        * `AppWatcher.cpp`, `AppWatcher.h` - 前台应用监控 [cite: 1]
    * `utils/` - 工具类模块 [cite: 1]
        * `Logger.cpp`, `Logger.h` - 日志记录 (基于spdlog) [cite: 1]
        * `ShellExecutor.cpp`, `ShellExecutor.h` - Shell命令执行，处理器型号识别辅助 [cite: 1]
        * `FileWatcher.cpp`, `FileWatcher.h` - 配置文件监控 (基于inotify) [cite: 1]
        * `DebugUtils.cpp`, `DebugUtils.h` - 调试标志文件工具 [cite: 1]
        * `CpuUtils.cpp`, `CpuUtils.h` - CPU列表解析和格式化工具（支持核心类型名称） [cite: 1]
    * `configs_template/` 或 `data/` - (可选) 默认配置文件模板存放目录，打包时应放入模块的 `configs/` 目录 [cite: 1]
        * `default_config.ini` [cite: 1]
        * `app_profiles.ini` [cite: 1]
        * `sdm8gen1.ini.example` (处理器特定配置示例) [cite: 1]
    * `third_party/` - 第三方库 [cite: 1]
        * `spdlog/` - spdlog 日志库 [cite: 1]
        * `inih/` - inih INI解析库 [cite: 1]
    * `magisk/` - (示例) Magisk模块打包所需文件及编译输出目标位置 [cite: 1]
        * `YukiCpuScheduler` - (示例) 编译后的可执行文件，应放置在模块的 `system/bin/` [cite: 1]
        * `customize.sh` - Magisk模块安装脚本 [cite: 1]
        * `post-fs-data.sh` - Magisk post-fs-data脚本 [cite: 1]
        * `service.sh` - Magisk服务启动脚本 [cite: 1]
        * `uninstall.sh` - Magisk卸载脚本 [cite: 1]
        * `module.prop` - Magisk模块属性文件 [cite: 1]
        * `sepolicy.rule` - SELinux策略规则 [cite: 1]
        * `file_contexts` - 文件SELinux上下文 [cite: 1]
        * `configs/` - **模块安装后，运行时配置文件的实际存放目录** [cite: 1]
            * `default_config.ini` [cite: 1]
            * `app_profiles.ini` [cite: 1]
            * (可选) `sdm8gen1.ini` 等处理器特定配置文件 [cite: 1]

---

## ⚙️ 如何手动配置 `default_config.ini`

本节将详细指导您如何修改 `default_config.ini` 文件（以及可选的处理器特定 `.ini` 文件和 `app_profiles.ini`）以自定义 YukiCpuScheduler 的行为。当存在与当前设备处理器型号匹配的特定配置文件（例如 `sdm8gen1.ini`，位于 `configs/` 目录下）时，该特定文件中的设置会**覆盖** `default_config.ini` 中的同名设置。同样，`app_profiles.ini` 中的设置会进一步覆盖当前生效模式的参数（针对特定应用）。

**重要提示:**

* **备份**: 在进行任何修改之前，强烈建议您备份原始的配置文件。 [cite: 1]
* **语法**:
    * 配置文件采用INI格式，由多个**节 (Section)** 组成，节名用方括号 `[]` 包围，例如 `[meta]`。 [cite: 1]
    * 每个节下面包含多个**键值对 (Key-Value Pairs)**，格式为 `键 = 值`，例如 `loglevel = DEBUG`。 [cite: 1]
    * **注释**: 以分号 `;` 或井号 `#` 开头的行为注释行，将被程序忽略。 [cite: 1]
    * **值格式**:
        * 除非特别说明，配置值**不应**包含英文双引号 `""` 或单引号 `''`。程序内部会处理首尾空白。 [cite: 1]
        * 布尔值通常使用 `true` 或 `false` (不区分大小写)。 [cite: 1]
        * 频率单位通常为 `kHz` (千赫兹)，例如 `1800000` 代表 1.8GHz。 [cite: 1]
        * 时间单位通常在键名中指明，例如 `Ms` (毫秒) 或 `Sec` (秒)。 [cite: 1]
* **重启服务/热重载**: 修改 `default_config.ini` 或 `app_profiles.ini` 后，服务会自动检测更改并热重载配置。如果修改了处理器特定配置文件，建议重启服务以确保更改完全生效（因为处理器型号通常在服务启动时确定一次）。 [cite: 1]

---

### 配置文件结构和主要配置项详细说明

#### 1. `[meta]` - 元数据和全局设置

本节包含服务的基础信息和全局行为控制。

* **`name`**: 字符串。模块的描述性名称，主要用于日志或未来可能的UI显示。用户可以根据喜好修改。 [cite: 1]
    * 示例: `name = "YukiCpuScheduler 增强版"`
* **`author`**: 字符串。模块作者的署名。 [cite: 1]
    * 示例: `author = "yuki"`
* **`configVersion`**: 整数。配置文件的版本号。主要供开发者用于追踪配置格式的演进，以便在未来版本中处理兼容性。用户通常不需要修改此项。 [cite: 1]
    * 示例: `configVersion = 27`
* **`ModuleRootPath`**: 字符串。【**非常重要**】模块在设备文件系统中的实际安装根目录的绝对路径。 [cite: 1]
    * 此路径是程序定位 `logs/` 目录、`debug_flags/` 目录等其他模块内部资源的基础。
    * 如果此项留空，程序会尝试通过分析日志目录参数或可执行文件的路径来推断。但为了确保准确性，强烈建议在部署模块时正确设置此路径。
    * 对于标准的 Magisk 模块，这通常是 `/data/adb/modules/YourModuleID/` (例如 `/data/adb/modules/YukiCpuScheduler`)。 [cite: 1]
    * 示例: `ModuleRootPath = "/data/adb/modules/YukiCpuScheduler"`
* **`loglevel`**: 字符串。定义服务输出到日志文件的日志信息的详细程度。 [cite: 1]
    * 可选值（从最详细到最不详细，不区分大小写）:
        * `TRACE`: 输出所有级别的日志，包括非常细致的追踪信息，主要用于深度调试。
        * `DEBUG`: 输出调试信息、信息、警告、错误和严重错误。适合开发和问题排查。
        * `INFO`: 输出常规操作信息、警告、错误和严重错误。适合日常使用以监控基本状态。
        * `WARN` (或 `WARNING`): 输出警告、错误和严重错误。
        * `ERROR` (或 `ERR`): 只输出错误和严重错误。
        * `CRITICAL`: 只输出最严重的错误。
        * `OFF`: 关闭所有日志记录。
    * 示例: `loglevel = INFO`
* **`screen_on_mode`**: 字符串。当屏幕点亮时，服务默认应用的调度模式的名称。 [cite: 1]
    * 这个名称必须对应在本配置文件后面“模式定义”部分中定义的一个模式节名（例如，如果定义了 `[balance]` 模式，这里就可以写 `balance`）。
    * 如果 `app_profiles.ini` 中为当前前台应用指定了不同的模式，则此默认值会被覆盖。
    * 示例: `screen_on_mode = balance`
* **`screen_off_mode`**: 字符串。当屏幕关闭时，服务默认应用的调度模式的名称。 [cite: 1]
    * 同样，需要对应一个已定义的模式节名。通常是一个省电模式。
    * 示例: `screen_off_mode = screenOffPowersave`
* **`EnablePerAppProfile`**: 布尔值 (`true` 或 `false`)。决定是否启用应用特定的配置。 [cite: 1]
    * 如果设为 `true`，服务会尝试加载 `app_profiles.ini` 文件（位于 `configs/` 目录下）。
    * 如果设为 `false`，则忽略 `app_profiles.ini` 文件。
    * 示例: `EnablePerAppProfile = true`

---

#### 2. `[monitor]` - 监控相关设置

本节控制服务如何以及以何种频率监控系统状态（如屏幕亮灭、前台应用变化）的变化。

* **`ScreenStateMethod`**: 字符串。指定检测屏幕点亮/熄灭状态所使用的方法。 [cite: 1]
    * `DUMPSYS_POWER`: 通过执行 `dumpsys power` 命令并解析其输出来获取屏幕状态。
    * `SYSFS`: 通过读取特定的sysfs文件节点来判断屏幕状态。
    * 示例: `ScreenStateMethod = DUMPSYS_POWER`
* **`ScreenStateSysfsPath`**: 字符串。仅当 `ScreenStateMethod = SYSFS` 时生效。 [cite: 1]
    * 指定用于判断屏幕状态的sysfs节点的绝对路径。例如 `/sys/class/leds/lcd-backlight/brightness`。
    * 示例: `ScreenStateSysfsPath = "/sys/class/leds/lcd-backlight/brightness"`
* **`ScreenCheckIntervalMs`**: 整数。屏幕状态检查的时间间隔，单位为毫秒。 [cite: 1]
    * 示例: `ScreenCheckIntervalMs = 2000`
* **`AppCheckIntervalMs`**: 整数。前台应用切换检查的时间间隔，单位为毫秒。 [cite: 1]
    * 示例: `AppCheckIntervalMs = 1000`
* **`MainLoopSleepSec`**: 整数。服务主循环的休眠时间，单位为秒。 [cite: 1]
    * 示例: `MainLoopSleepSec = 5`
* **`EnterStandbyDelayMs`**: 整数。当屏幕关闭后，延迟多久服务进入定义的“待机模式”，单位为毫秒。 [cite: 1]
    * 在此待机模式下，`AppWatcher` 会暂停。
    * 示例: `EnterStandbyDelayMs = 300000` (5分钟)

---

#### 3. `[monitor_tuning]` - 监控调优参数

本节用于微调服务在启动初期探测依赖服务以及常规监控时执行 `dumpsys` 命令的参数。

* **`InitialProbeMaxRetries`**: 整数。服务启动时，对依赖的系统服务进行就绪状态探测的最大重试次数。 [cite: 1]
* **`InitialProbeDelayMs`**: 整数。每次初始服务就绪探测尝试之间的延迟（毫秒）。 [cite: 1]
* **`DumpsysPowerProbeCommand`**: 字符串。用于探测 `PowerManagerService` 是否就绪的 `dumpsys` 命令。 [cite: 1]
* **`DumpsysPowerMonitorCommand`**: 字符串。用于常规监控屏幕状态的 `dumpsys power` 命令。 [cite: 1]
* **`DumpsysPowerMonitorRetries`**: 整数。常规监控时 `dumpsys power` 命令失败后的重试次数。 [cite: 1]
* **`DumpsysPowerMonitorDelayMs`**: 整数。常规监控时 `dumpsys power` 命令失败后的重试延迟（毫秒）。 [cite: 1]
* **`DumpsysActivityProbeCommand`**: 字符串。用于探测 `ActivityManagerService` 是否就绪的 `dumpsys` 命令。 [cite: 1]
* **`DumpsysActivityMonitorCommand`**: 字符串。用于常规监控前台应用的 `dumpsys activity` 命令。 [cite: 1]
* **`DumpsysActivityMonitorRetries`**: 整数。常规监控时 `dumpsys activity` 命令失败后的重试次数。 [cite: 1]
* **`DumpsysActivityMonitorDelayMs`**: 整数。常规监控时 `dumpsys activity` 命令失败后的重试延迟（毫秒）。 [cite: 1]

---

#### 4. `[function]` - 主要功能开关

本节包含一系列布尔值 (`true` 或 `false`)，用于启用或禁用 YukiCpuScheduler 的各项核心优化功能。 [cite: 1]

* **`CpuFreqGovControl`**: 控制CPU频率和调速器。 [cite: 1]
* **`UclampControl`**: 控制 cgroup 的 Uclamp 值。 [cite: 1]
* **`CpusetControl`**: 控制 cgroup 的 Cpuset 核心绑定。 [cite: 1]
* **`AffinitySetter`**: 为特定cgroup内的进程设置CPU亲和性。 [cite: 1]
* **`AdjIOScheduler`**: 调整块设备的I/O调度器。 [cite: 1]
* **`IO_optimization`**: 进行其他I/O优化 (如nomerges)。 [cite: 1]
* **`EasScheduler`**: 设置EAS相关的内核参数。 [cite: 1]
* **`AppLaunchBoost`**: 启用应用启动加速功能。 [cite: 1]

---

#### 5. `[CoreFramework]` - CPU核心框架路径配置

定义不同CPU核心簇的`cpufreq`驱动的代表性路径，并影响核心类型的自动检测与映射。

* **`SmallCorePath`**, **`MediumCorePath`**, **`BigCorePath`**, **`SuperBigCorePath`**: [cite: 1]
    * 值可以是 `auto` (推荐，程序自动检测), CPU索引号 (如 `0`), 完整cpufreq路径 (如 `/sys/devices/system/cpu/cpu0/cpufreq`), 或 `-1` (禁用此类型)。 [cite: 1]
    * **自动模式 (`auto`)**：程序会基于 `cpuinfo_max_freq` 识别核心簇并分配合适的逻辑类型名（如SmallCore, BigCore等）。如果未能识别出所有四种标准类型（例如设备没有超大核），程序会跳过未找到的类型而不会报错。
    * **手动指定模式**：如果用户为某个核心类型（如 `BigCorePath`）手动指定了路径或CPU索引，程序会尝试验证该路径。若路径无效，则会记录错误，该核心类型将无代表性调频路径。
    * 示例: `SmallCorePath = auto`

---

#### 6. `[IO_Settings]` - I/O调度相关设置

* **`BlockDevice`**: 目标块设备名称 (`auto` 或手动指定如 `sda`)。 [cite: 1]
* **`Scheduler`**: 期望设置的I/O调度器名称 (`bfq`, `none` 等)。 [cite: 1]
* **`ReadAheadKb`**: (可选) 块设备的预读大小（KB）。 [cite: 1]

---

#### 7. `[EasSettings]` 和 `[EasSchedulerValue]` - EAS参数设置

调整内核Energy Aware Scheduling (EAS) 相关参数。

* **`[EasSettings]` `BasePaths`**: 逗号分隔的EAS参数基础路径列表。 [cite: 1]
* **`[EasSchedulerValue]`**: 定义要设置的EAS参数及其值 (例如 `sched_energy_aware = 1`)。 [cite: 1]

---

#### 8. `[CgroupPaths]` - Cgroup路径定义

定义各类cgroup的文件系统路径或路径前缀。

* **`XxxCpuCtlPathPrefix`**: Uclamp设置路径前缀 (例如 `TopAppCpuCtlPathPrefix = "/dev/cpuctl/top-app"`)。 [cite: 1]
* **`XxxCpusetPath`**: Cpuset设置路径 (例如 `BackgroundCpusetPath = "/dev/cpuset/background"`)。 [cite: 1]

---

#### 9. `[CpusetSettings]` - 默认Cpuset核心绑定

为cgroup类别设置默认CPU核心绑定。

* 键: cgroup类别名 (如 `TopApp`, `Background`)。
* 值: CPU核心列表字符串。**支持核心类型名称** (如 "SmallCore", "BigCore", "SuperBigCore"), 特殊关键字 (如 "SmallestSingle", "AllCores"), 以及传统数字和范围 (如 "0-3", "7")。 [cite: 1, 2]
    * 程序会根据 `[CoreFramework]` 的检测结果动态解析核心类型名称。 [cite: 1, 2]
    * 若配置中指定的核心类型名称在当前设备未检测到，该名称将被**跳过**。 [cite: 1]
    * 示例: `TopApp = "MediumCore,BigCore,SuperBigCore"` [cite: 2]

---

#### 10. `[AffinitySettings]` - CPU亲和性设置

为cgroup类别（或结合模式）设置CPU亲和性。

* 格式与值的解析逻辑同 `[CpusetSettings]` (同样支持核心类型名称和跳过逻辑)。 [cite: 1]
    * `Affinity_CgroupCategoryKey = "核心列表字符串"` [cite: 1, 2]
    * `Affinity_CgroupCategoryKey_ModeName = "核心列表字符串"` [cite: 1, 2]
    * 示例: `Affinity_Background_powersave = "SmallestSingle"` [cite: 2]

---

#### 11. `[AppLaunchBoostSettings]` - 应用启动加速设置

控制应用启动加速功能。

* **`Enable`**: 是否启用 (`true`/`false`)。 [cite: 1, 2]
* **`BoostDurationMs`**: 加速持续时间（毫秒）。 [cite: 1, 2]
* **`BoostModeName`**: 加速时应用的调度模式。 [cite: 1, 2]
* **`CoolDownMs`**: 冷却时间（毫秒）。 [cite: 1, 2]
* **`ExcludedApps`**: 逗号分隔的不触发加速的应用包名列表。 [cite: 1, 2]

---

#### 12. 模式定义 (例如 `[powersave]`, `[balance]`, `[performance]`)

定义不同场景下的系统行为。每个模式都是一个独立的节。

* **在一个模式节内:**
    * **主调速器与备选调速器**:
        * `ScalingGovernor = governor_name` (例如 `schedutil`)：此模式下所有核心簇的默认主调速器。
        * `FallbackScalingGovernor = governor_name` (例如 `interactive`)：如果主调速器设置失败，则尝试此备选调速器。
    * **针对特定核心簇的参数**: (前缀 `SmallCore`, `MediumCore`, `BigCore`, `SuperBigCore`)
        * `CoreTypeScalingGovernor = governor_name`: 该核心簇的主调速器。
        * `CoreTypeFallbackScalingGovernor = governor_name`: 该核心簇的备选调速器。
        * `CoreTypeMaxFreq = frequency_in_kHz | max`: 最大频率。 [cite: 1, 2]
        * `CoreTypeMinFreq = frequency_in_kHz | min`: 最小频率。 [cite: 1, 2]
        * **Schedutil 特定参数**: (仅当该核心簇**最终生效**的调速器是 `schedutil` 时应用)
            * `CoreTypeSchedutilUpRateLimitUs = value_in_us` [cite: 1, 2]
            * `CoreTypeSchedutilDownRateLimitUs = value_in_us` [cite: 1, 2]
            * 其他如 `CoreTypeSchedutilHispeedLoad`, `CoreTypeSchedutilHispeedFreq` 等。 [cite: 1]
        * **WALT 特定参数 (新增)**: (仅当该核心簇的 `cpufreq` 目录下存在 `walt/` 子目录时应用)
            * 键名格式: `CoreTypeWaltParameterNameInCamelCase` (例如 `SmallCoreWaltHispeedFreq`)。程序会将 `ParameterNameInCamelCase` 转换为下划线形式的节点名。
            * 示例: `SmallCoreWaltUpRateLimitUs = 500`
        * **UAG 特定参数 (新增)**: (仅当该核心簇的 `cpufreq` 目录下存在 `uag/` 子目录时应用)
            * 键名格式: `CoreTypeUagParameterNameInCamelCase` (例如 `BigCoreUagStallAware`)。
            * 示例: `BigCoreUagTargetLoads = "80 2000000 90"`
    * **模式全局Uclamp设置**: (键名中的 `CgroupCategoryKey` 替换为 `TopApp` 等) [cite: 1, 2]
        * `UclampMin_CgroupCategoryKey = value`
        * `UclampMax_CgroupCategoryKey = value`
        * `UclampLatencySensitive_CgroupCategoryKey = 0_or_1`
    * **模式特定的 Cpuset/Affinity 设置 (可选)**:
        * `Cpuset_CgroupCategoryKey = "核心列表字符串"` (覆盖 `[CpusetSettings]` 中的同名项)
        * `Affinity_CgroupCategoryKey = "核心列表字符串"` (覆盖 `[AffinitySettings]` 中的同名项)
    * **直接写入Sysfs节点**: `SysfsPath_/path/to/sysfs/node = value_to_write` (路径必须是绝对路径) [cite: 1]

---

## 🤝 贡献 (Contributing)

欢迎通过 Pull Requests 贡献代码，或在 Issues 中报告问题和提出建议。

## 📜 许可证 (License)

本项目采用 MIT 许可证。
