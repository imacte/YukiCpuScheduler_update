# YukiCpuScheduler (Yuki CPU 调度器)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**YukiCpuScheduler 是一个为 Android 设备设计的、高度可配置的后台服务，旨在通过动态调整CPU频率、调速器、Uclamp、Cpuset、CPU亲和性、I/O调度器和EAS（如果内核支持）参数，来优化设备性能和功耗表现。**

它特别关注于根据当前运行的前台应用和屏幕状态来应用不同的调度策略，以实现在需要高性能时（如游戏、应用启动）提供流畅体验，在日常使用中保持均衡，在设备空闲或息屏时则尽可能节省电量。

---

## ✨ 主要特性 (Features)

* **多模式调度：** 支持预定义多种性能模式（例如 `powersave`, `balance`, `performance`, `fast`），并可根据屏幕状态自动切换。 [cite: 1]
* **应用特定配置：** 允许用户为特定的应用程序（通过包名识别）定义专属的调度模式或覆盖模式中的个别参数。 [cite: 1]
* **全面的CPU参数调整：**
    * 支持多核心簇架构 (例如 Small, Medium, Big, SuperBig Cores)。 [cite: 1]
    * 可配置每个核心簇的 CPU 调速器 (governor)。 [cite: 1]
    * 可配置每个核心簇的最大和最小 CPU 频率。 [cite: 1]
    * 可配置特定调速器参数 (例如 `schedutil` 的 `up_rate_limit_us`, `down_rate_limit_us` 等)。 [cite: 1]
* **Uclamp 控制：** 根据模式和应用调整 cgroup 的 `cpu.uclamp.min`, `cpu.uclamp.max` 和 `cpu.uclamp.latency_sensitive` (如果内核支持 `cpu.uclamp.latency_sensitive` 或类似的 `cpu.latency_sensitive`)，为调度器提供性能需求提示。 [cite: 1]
* **Cpuset 控制：** 为不同的进程组 (cgroups 如 top-app, background) 分配特定的 CPU 核心集。支持使用核心类型名称（如 "SmallCore", "BigCore"）进行配置。 [cite: 1]
* **CPU 亲和性设置：** 为特定 cgroup 内的进程设置 CPU 亲和性，将它们绑定到指定的 CPU 核心上。支持使用核心类型名称。 [cite: 1]
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

YukiCpuScheduler 主要通过以下 INI 格式的配置文件进行配置，它们通常位于 Magisk 模块的 `/data/adb/modules/YukiCpuScheduler/configs/` 目录 (或者你在 CMakeLists.txt 中指定的模块路径，或服务启动时通过参数指定的路径)：

1.  **`default_config.ini`**: 主配置文件，定义了全局设置、功能开关、默认模式、各模式下的详细参数、cgroup路径、EAS参数等。 [cite: 1]
2.  **`app_profiles.ini`**: 应用特定配置文件，用于为指定的应用程序覆盖默认模式或模式中的参数。 [cite: 1]
3.  **处理器特定配置文件 (可选)**: 例如 `sdm8gen1.ini`。如果存在与当前设备处理器型号匹配的此类文件，其配置将覆盖 `default_config.ini` 中的同名设置。

详细的配置项说明和示例请参考这些文件内的注释，或参阅下方的“如何手动配置 default_config.ini”章节。

---

## 🚀 如何使用 (Usage)

1.  **编译：**
    * 确保你已安装 Android NDK (例如 r23c 或更新版本) 并设置了 `ANDROID_NDK_HOME` 环境变量。 [cite: 1]
    * 安装 CMake (例如 3.10 或更新版本) 和编译工具链 (例如 `build-essential` 在 Debian/Ubuntu 上)。 [cite: 1]
    * 在项目根目录下创建 `build` 目录并进入： [cite: 1]
        ```bash
        mkdir build && cd build
        ```
    * 运行 CMake 配置命令，指定工具链文件和目标ABI： [cite: 1]
        ```bash
        cmake .. \
            -DCMAKE_TOOLCHAIN_FILE="$ANDROID_NDK_HOME/build/cmake/android.toolchain.cmake" \
            -DANDROID_ABI="arm64-v8a" \
            -DANDROID_PLATFORM="android-24" \ # 或更高API级别
            -DCMAKE_BUILD_TYPE="Release"      # 或 "Debug"
        ```
    * 编译项目： [cite: 1]
        ```bash
        make -j$(nproc)
        ```
    * 生成的可执行文件 `YukiCpuScheduler` 会在 CMakeLists.txt 中 `CMAKE_RUNTIME_OUTPUT_DIRECTORY` 指定的输出目录 (例如您配置的 `YukiCpuScheduler/magisk/`)。 [cite: 1]

2.  **部署 (作为 Magisk 模块示例)：**
    * 将编译好的 `YukiCpuScheduler`可执行文件、`configs/default_config.ini`、`configs/app_profiles.ini` (以及可能的处理器特定ini文件) 和其他模块脚本（如 `service.sh`, `post-fs-data.sh`, `module.prop`, `sepolicy.rule`）打包成一个 Magisk 模块ZIP文件。 [cite: 1]
    * `service.sh` 脚本负责在系统启动时以 root 权限启动 `YukiCpuScheduler` 可执行文件，并传递正确的配置文件路径。 [cite: 1]
    * `sepolicy.rule` 定义了服务运行所需的 SELinux 权限。 [cite: 1]
    * 确保您的 Magisk 模块ZIP包结构正确，以便配置文件能被放置到模块安装后的 `configs/` 目录下。 [cite: 1]

3.  **配置：**
    * 根据你的设备和需求，仔细编辑位于模块 `configs/` 目录下的 `default_config.ini` 和 `app_profiles.ini`。 [cite: 1]
    * 如果需要为特定处理器进行深度定制，可以创建对应的处理器特定配置文件 (如 `sdm8gen1.ini`) 并放置在 `configs/` 目录下。 [cite: 1]
    * 特别注意 `[CoreFramework]` 中的核心路径配置，以及各个模式下的频率、调速器和Uclamp参数。 [cite: 1]
    * 参阅下方的“如何手动配置 default_config.ini”章节获取详细指导。 [cite: 1]

4.  **日志和调试：**
    * 服务的日志文件 (`scheduler.log`) 默认存放在模块的 `logs` 目录下 (路径可通过命令行参数 `--log-dir` 或配置文件中 `ModuleRootPath` 推断)。 [cite: 1]
    * `service.sh` 和 `post-fs-data.sh` 的执行日志也存放在模块的 `logs` 目录下。 [cite: 1]
    * 模块根目录下的 `debug_flags` 目录中可能会创建调试标志文件。 [cite: 1]
    * 可以通过修改 `default_config.ini` 中 `[meta] loglevel` 来调整日志详细程度。 [cite: 1]

---

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

本节将详细指导您如何修改 `default_config.ini` 文件以自定义 YukiCpuScheduler 的行为。当存在与当前设备处理器型号匹配的特定配置文件（例如 `sdm8gen1.ini`，位于 `configs/` 目录下）时，该特定文件中的设置会覆盖 `default_config.ini` 中的同名设置。

**重要提示:**

* **备份**: 在进行任何修改之前，强烈建议您备份原始的配置文件。 [cite: 1]
* **语法**:
    * 配置文件采用INI格式，由多个**节 (Section)** 组成，节名用方括号 `[]` 包围，例如 `[meta]`。 [cite: 1]
    * 每个节下面包含多个**键值对 (Key-Value Pairs)**，格式为 `键 = 值`，例如 `loglevel = DEBUG`。 [cite: 1]
    * **注释**: 以分号 `;` 或井号 `#` 开头的行为注释行，将被程序忽略。 [cite: 1]
    * **值格式**:
        * 除非特别说明，配置值**不应**包含英文双引号 `""` 或单引号 `''`。程序内部会处理首尾空白。 [cite: 1]
        * 布尔值通常使用 `true` 或 `false`。 [cite: 1]
        * 频率单位通常为 `kHz` (千赫兹)，例如 `1800000` 代表 1.8GHz。 [cite: 1]
        * 时间单位通常在键名中指明，例如 `Ms` (毫秒) 或 `Sec` (秒)。 [cite: 1]
* **重启服务/热重载**: 修改 `default_config.ini` 或 `app_profiles.ini` 后，服务会自动检测更改并热重载配置。如果修改了处理器特定配置文件，建议重启服务以确保更改完全生效。 [cite: 1]

---

### 配置文件结构和主要配置项详细说明

#### 1. `[meta]` - 元数据和全局设置

本节包含服务的基础信息和全局行为控制。

* **`name`**: 字符串。模块的描述性名称，主要用于日志或未来可能的UI显示。用户可以根据喜好修改。 [cite: 1]
    * 示例: `name = "YukiCpuScheduler 增强版"`
* **`author`**: 字符串。模块作者的署名。 [cite: 1]
    * 示例: `author = "yuki"`
* **`configVersion`**: 整数。配置文件的版本号。主要供开发者用于追踪配置格式的演进，以便在未来版本中处理兼容性。用户通常不需要修改此项。 [cite: 1]
    * 示例: `configVersion = 26`
* **`ModuleRootPath`**: 字符串。【**非常重要**】模块在设备文件系统中的实际安装根目录的绝对路径。 [cite: 1]
    * 此路径是程序定位 `logs/` 目录、`debug_flags/` 目录等其他模块内部资源的基础。
    * 如果此项留空，程序会尝试通过分析日志目录参数或可执行文件的路径来推断。但为了确保准确性，强烈建议在部署模块时正确设置此路径。
    * 对于标准的 Magisk 模块，这通常是 `/data/adb/modules/YourModuleID/`。 [cite: 1]
    * 示例: `ModuleRootPath = "/data/adb/modules/YukiCpuScheduler"`
* **`loglevel`**: 字符串。定义服务输出到日志文件的日志信息的详细程度。 [cite: 1]
    * 可选值（从最详细到最不详细）:
        * `TRACE`: 输出所有级别的日志，包括非常细致的追踪信息，主要用于深度调试。
        * `DEBUG`: 输出调试信息、信息、警告、错误和严重错误。适合开发和问题排查。
        * `INFO`: 输出常规操作信息、警告、错误和严重错误。适合日常使用以监控基本状态。
        * `WARN`: 输出警告、错误和严重错误。
        * `ERROR`: 只输出错误和严重错误。
        * `CRITICAL`: 只输出最严重的错误。
        * `OFF`: 关闭所有日志记录。
    * 示例: `loglevel = INFO`
* **`screen_on_mode`**: 字符串。当屏幕点亮时，服务默认应用的调度模式的名称。 [cite: 1]
    * 这个名称必须对应在本配置文件后面“模式定义”部分中定义的一个模式节名（例如，如果定义了 `[balance]` 模式，这里就可以写 `balance`）。
    * 如果应用了特定于应用的配置文件 (`app_profiles.ini`) 并且该应用有自己的模式设置，则此默认值可能被覆盖。
    * 示例: `screen_on_mode = balance`
* **`screen_off_mode`**: 字符串。当屏幕关闭时，服务默认应用的调度模式的名称。 [cite: 1]
    * 同样，需要对应一个已定义的模式节名。通常是一个省电模式。
    * 示例: `screen_off_mode = screenOffPowersave`
* **`EnablePerAppProfile`**: 布尔值 (`true` 或 `false`)。决定是否启用应用特定的配置。 [cite: 1]
    * 如果设为 `true`，服务会尝试加载 `app_profiles.ini` 文件（位于 `configs/` 目录下）。该文件允许为不同的应用程序（通过其包名识别）指定不同的调度模式或覆盖当前模式下的特定参数。
    * 如果设为 `false`，则忽略 `app_profiles.ini` 文件。
    * 示例: `EnablePerAppProfile = true`

---

#### 2. `[monitor]` - 监控相关设置

本节控制服务如何以及以何种频率监控系统状态（如屏幕亮灭、前台应用变化）的变化。

* **`ScreenStateMethod`**: 字符串。指定检测屏幕点亮/熄灭状态所使用的方法。 [cite: 1]
    * `DUMPSYS_POWER`: 通过执行 `dumpsys power` 命令并解析其输出来获取屏幕状态。这种方法具有较好的跨设备通用性，但执行 `dumpsys` 命令本身有一定开销，且在系统启动的极早期阶段，`dumpsys` 服务可能尚未完全就绪。
    * `SYSFS`: 通过读取特定的sysfs文件节点（通常是屏幕背光亮度节点或某些显示控制器状态节点）来判断屏幕状态。这种方法通常速度更快、资源消耗更低，但缺点是sysfs节点路径可能因设备型号、ROM版本甚至内核版本的不同而有所差异，需要用户自行查找并确认。
    * 示例: `ScreenStateMethod = DUMPSYS_POWER`
* **`ScreenStateSysfsPath`**: 字符串。仅当 `ScreenStateMethod` 设置为 `SYSFS` 时生效。 [cite: 1]
    * 指定用于判断屏幕状态的sysfs节点的绝对路径。例如，很多设备的LCD背光亮度节点路径类似于 `/sys/class/leds/lcd-backlight/brightness`。程序会读取此节点的值，如果大于0则认为屏幕是亮的。某些设备可能有其他特定节点表示屏幕开关状态（例如，值为 "0" 代表亮，"1" 或更高代表灭，或者 "on"/"off" 字符串）。
    * 示例: `ScreenStateSysfsPath = "/sys/class/leds/lcd-backlight/brightness"`
* **`ScreenCheckIntervalMs`**: 整数。屏幕状态检查的时间间隔，单位为毫秒。程序会按此频率检查屏幕是否点亮或熄灭。 [cite: 1]
    * 示例: `ScreenCheckIntervalMs = 2000` (即每2秒检查一次)
* **`AppCheckIntervalMs`**: 整数。前台应用切换检查的时间间隔，单位为毫秒。程序会按此频率获取当前的前台应用包名。 [cite: 1]
    * 示例: `AppCheckIntervalMs = 1000` (即每1秒检查一次)
* **`MainLoopSleepSec`**: 整数。服务主循环（在完成所有初始化和启动所有监控线程后）的休眠时间，单位为秒。这个循环主要用于保持服务存活并定期打印心跳日志。 [cite: 1]
    * 示例: `MainLoopSleepSec = 5`
* **`EnterStandbyDelayMs`**: 整数。当屏幕关闭后，延迟多久系统才真正进入由本服务定义的“待机模式”（注意，这可能与Android系统的Doze或Standby模式不完全相同），单位为毫秒。 [cite: 1]
    * 在本服务的“待机模式”下，通常会暂停一些活动（如 `AppWatcher` 的轮询）以进一步降低功耗。
    * 如果设置为 `0`，则屏幕关闭后会立即尝试进入此待机状态（如果 `screen_off_mode` 本身不是设计为极度省电的话）。
    * 示例: `EnterStandbyDelayMs = 300000` (即5分钟)

---

#### 3. `[monitor_tuning]` - 监控调优参数

本节用于微调服务在启动初期探测依赖的系统服务（如 `PowerManagerService`, `ActivityManagerService`）时的行为参数，以及在常规监控过程中执行相关 `dumpsys` 命令的参数。

* **`InitialProbeMaxRetries`**: 整数。服务启动时，对每个依赖的系统服务（如电源管理、活动管理）进行就绪状态探测的最大重试次数。如果设为 `0`，则跳过启动时的服务就绪探测，直接假设服务可用。 [cite: 1]
    * 示例: `InitialProbeMaxRetries = 12`
* **`InitialProbeDelayMs`**: 整数。每次初始服务就绪探测尝试之间的延迟时间，单位为毫秒。 [cite: 1]
    * 示例: `InitialProbeDelayMs = 5000`
* **`DumpsysPowerProbeCommand`**: 字符串。用于在服务启动初期**探测** `PowerManagerService` 是否就绪的轻量级 `dumpsys` 命令。该命令的输出应能间接或直接表明服务是否可用。 [cite: 1]
    * 示例: `DumpsysPowerProbeCommand = "dumpsys power properties"`
* **`DumpsysPowerMonitorCommand`**: 字符串。用于在服务正常运行时**常规监控**屏幕状态（当 `ScreenStateMethod` 为 `DUMPSYS_POWER` 时）的 `dumpsys power` 命令。 [cite: 1]
    * 示例: `DumpsysPowerMonitorCommand = "dumpsys power"`
* **`DumpsysPowerMonitorRetries`**: 整数。在常规监控屏幕状态时，如果单次执行 `DumpsysPowerMonitorCommand` 命令失败（例如 `dumpsys` 暂时无响应），程序会进行的重试次数。 [cite: 1]
    * 示例: `DumpsysPowerMonitorRetries = 2`
* **`DumpsysPowerMonitorDelayMs`**: 整数。在常规监控屏幕状态时，`DumpsysPowerMonitorCommand` 命令执行失败后的重试延迟时间，单位为毫秒。 [cite: 1]
    * 示例: `DumpsysPowerMonitorDelayMs = 300`
* **`DumpsysActivityProbeCommand`**: 字符串。用于在服务启动初期**探测** `ActivityManagerService` 是否就绪的 `dumpsys` 命令。 [cite: 1]
    * 示例: `DumpsysActivityProbeCommand = "dumpsys activity brief"`
* **`DumpsysActivityMonitorCommand`**: 字符串。用于在服务正常运行时**常规监控**当前前台应用的 `dumpsys activity` 命令。 `AppWatcher` 会解析此命令的输出来获取前台应用包名。 [cite: 1]
    * 示例: `DumpsysActivityMonitorCommand = "dumpsys activity activities"`
* **`DumpsysActivityMonitorRetries`**: 整数。在常规监控前台应用时，如果单次执行 `DumpsysActivityMonitorCommand` 命令失败，程序会进行的重试次数。 [cite: 1]
    * 示例: `DumpsysActivityMonitorRetries = 2`
* **`DumpsysActivityMonitorDelayMs`**: 整数。在常规监控前台应用时，`DumpsysActivityMonitorCommand` 命令执行失败后的重试延迟时间，单位为毫秒。 [cite: 1]
    * 示例: `DumpsysActivityMonitorDelayMs = 250`

---

#### 4. `[function]` - 主要功能开关

本节包含一系列布尔值 (`true` 或 `false`)，用于启用或禁用 YukiCpuScheduler 的各项核心优化功能。 [cite: 1]

* **`CpuFreqGovControl`**: 是否允许服务控制CPU频率和调速器 (governor)。如果为 `false`，则 `Scheduler::apply_mode` 中所有与设置 `scaling_governor`, `scaling_max_freq`, `scaling_min_freq` 以及特定调速器参数（如 schedutil 的 `rate_limit_us` 等）相关的逻辑都将被跳过。 [cite: 1]
* **`UclampControl`**: 是否允许服务控制 cgroup 的 Uclamp 值（`cpu.uclamp.min`, `cpu.uclamp.max`, `cpu.uclamp.latency_sensitive`）。如果为 `false`，则 `CgroupManager::apply_uclamp_settings` 将被跳过。 [cite: 1]
* **`CpusetControl`**: 是否允许服务控制 cgroup 的 Cpuset（将任务绑定到特定CPU核心集）。如果为 `false`，则 `CgroupManager::apply_cpuset_settings` 将被跳过。 [cite: 1]
* **`AffinitySetter`**: 是否允许服务为特定 cgroup 内的进程设置 CPU 亲和性。如果为 `false`，则 `AffinityManager::apply_affinity_settings` 将被跳过。 [cite: 1]
* **`AdjIOScheduler`**: 是否允许服务调整块设备的 I/O 调度器。如果为 `false`，则 `IoManager` 中设置 I/O 调度器的逻辑将被跳过。 [cite: 1]
* **`IO_optimization`**: 是否允许服务进行其他 I/O 优化（例如，在 `IoManager` 中可能实现的调整 `nomerges` 等参数）。 [cite: 1]
* **`EasScheduler`**: 是否允许服务设置与 EAS (Energy Aware Scheduling) 相关的内核参数。如果为 `false`，则 `EasManager::set_eas_parameter` 将被跳过。 [cite: 1]
* **`AppLaunchBoost`**: 是否启用应用启动加速功能。如果为 `false`，则 `AppLaunchManager` 的相关逻辑将被跳过。 [cite: 1]
    * 示例: `CpuFreqGovControl = true`

---

#### 5. `[CoreFramework]` - CPU核心框架路径配置

本节定义不同CPU核心簇（小核、中核、大核、超大核，这些是程序内部的逻辑分类名称）的 `cpufreq` 驱动的代表性路径。这些路径用于 `Scheduler` 模块读取和写入该核心簇的频率、调速器等参数。
同时，**此处的配置（特别是当设为 `auto` 时）会影响 `Scheduler` 如何通过自动检测来识别和映射物理CPU核心到这些逻辑核心类型 (SmallCore, MediumCore, BigCore, SuperBigCore)。这些识别出的类型名称及其包含的CPU ID列表，可用于后续的 `[CpusetSettings]` 和 `[AffinitySettings]` 配置中通过核心类型名称来指定CPU核心。** [cite: 1]

* **`SmallCorePath`**, **`MediumCorePath`**, **`BigCorePath`**, **`SuperBigCorePath`**:
    * 值为 `auto`: 【**推荐**】程序将尝试根据每个CPU核心的 `cpuinfo_max_freq` (最大频率) 自动检测不同的核心簇，并为它们分配合适的逻辑类型名称 (SmallCore, MediumCore, BigCore, SuperBigCore，根据频率高低和簇的数量决定) 及代表性调频路径。如果在自动检测模式下，某种预期的核心类型（例如设备上不存在超大核，则 SuperBigCore）没有被检测到，程序会**静默跳过**该类型，不会报错。只要至少有一个核心簇被识别，自动检测就会尝试进行。
    * 值为CPU索引号 (例如 `0`, `4`, `7`): 代表该逻辑核心类型使用编号为X的CPU核心的 `cpufreq` 路径作为代表。程序会自动拼接成如 `/sys/devices/system/cpu/cpu0/cpufreq` 的路径。
    * 值为完整的cpufreq路径 (例如 `/sys/devices/system/cpu/cpu0/cpufreq` 或 `/sys/devices/system/cpu/policy0`——注意 `policyX` 路径下直接就是调速器等节点，不再有 `cpufreq` 子目录)。
    * 值为 `-1`: 表示禁用对该逻辑核心类型的管理，程序将不会为该核心类型设置任何调频参数，并且在代表性路径映射中也会忽略此类型。
    * **手动指定路径的错误处理**：如果 `[CoreFramework]` 中至少有一项不是 `auto` 或 `-1`（即存在手动指定的路径或索引），而该手动指定的路径或索引最终被验证为无效（例如路径不存在，或对应的 `scaling_governor` 节点不可访问），程序会在日志中**记录一个错误**，并且该核心类型将不会有代表性调频路径被设置。
    * 示例:
        ```ini
        SmallCorePath = auto
        MediumCorePath = auto
        BigCorePath = auto
        SuperBigCorePath = auto 
        ```

---

#### 6. `[IO_Settings]` - I/O调度相关设置

* **`BlockDevice`**: 字符串。目标块设备的名称。 [cite: 1]
    * `auto`: 程序将尝试自动检测系统主要的存储块设备（例如通过查找 `/sys/block` 下常见的 `sda`, `sdb`, `mmcblk0` 等，并验证其是否具有 `queue/scheduler` 节点）。
    * 也可以手动指定确切的块设备名称，例如 `sda` 或 `mmcblk0`。
    * 示例: `BlockDevice = auto`
* **`Scheduler`**: 字符串。期望为上述块设备设置的I/O调度器名称。 [cite: 1]
    * 可选值取决于您的内核支持，常见的有 `bfq`, `kyber`, `mq-deadline`, `cfq` (较老内核), `noop`, `deadline`。
    * 如果设为 `none` (不区分大小写)，则程序不会尝试修改当前的I/O调度器。
    * 示例: `Scheduler = bfq`
* **`ReadAheadKb`**: 整数 (可选)。块设备的预读大小，单位为KB。如果需要调整，可以取消注释并设置一个合适的值。通常内核有默认值。
    * 示例: `;ReadAheadKb = 128` (此行为注释状态，表示使用系统默认)

---

#### 7. `[EasSettings]` 和 `[EasSchedulerValue]` - EAS参数设置

用于调整内核的Energy Aware Scheduling (EAS) 相关参数，前提是您的内核支持EAS并且暴露了相应的sysfs控制节点。

* **`[EasSettings]`**
    * **`BasePaths`**: 字符串，一个**逗号分隔**的基础路径列表。当设置 `[EasSchedulerValue]` 中的参数时，程序会依次尝试在这些基础路径下寻找对应的参数文件名。 [cite: 1]
        * 例如，如果 `BasePaths = "/proc/sys/kernel/,/sys/kernel/sched/"` 且 `[EasSchedulerValue]` 中有 `my_eas_param = 1`，程序会先尝试写入 `/proc/sys/kernel/my_eas_param`，如果失败或找不到，再尝试 `/sys/kernel/sched/my_eas_param`。
        * 示例: `BasePaths = "/proc/sys/kernel/,/sys/kernel/sched/,/sys/devices/system/cpu/eas/"`
* **`[EasSchedulerValue]`**: 此节下的每一行都是一个 `参数名 = 值` 的键值对。 [cite: 1]
    * **参数名**：应该是实际的sysfs文件名，**不应**包含基础路径部分。
    * **值**：要写入该sysfs节点的值。
    * 示例:
        ```ini
        sched_energy_aware = 1
        sched_schedstats = 0 
        ; some_other_eas_knob = some_value
        ```

---

#### 8. `[CgroupPaths]` - Cgroup路径定义

定义了服务需要操作的各个cgroup类别（如前台应用、后台应用等）对应的文件系统路径或路径前缀。这些路径对于 `UclampControl`, `CpusetControl`, `AffinitySetter` 等功能至关重要。

* **`XxxCpuCtlPathPrefix`**: 字符串。用于Uclamp设置的cgroup路径前缀。程序通常会在此路径下拼接上具体的Uclamp文件名（如 `cpu.uclamp.min`, `cpu.uclamp.max`）。 [cite: 1]
    * `Xxx` 代表cgroup类别键，例如 `TopApp`, `Foreground`, `Background`, `SystemBg` (或在代码中映射为 `SystemBackground`), `Restricted`。
    * 示例: `TopAppCpuCtlPathPrefix = "/dev/cpuctl/top-app"`
* **`XxxCpusetPath`**: 字符串。用于Cpuset设置和AffinityManager获取PID列表的cgroup路径。程序通常会在此路径下操作 `cpus`（设置核心绑定）和 `tasks`（或 `cgroup.procs`，迁移进程）文件。 [cite: 1]
    * `Xxx` 的含义同上。
    * 示例: `BackgroundCpusetPath = "/dev/cpuset/background"`

---

#### 9. `[CpusetSettings]` - 默认Cpuset核心绑定

为不同的cgroup类别设置默认的CPU核心绑定。这些设置会被 `CgroupManager::apply_cpuset_settings` 读取和应用。

* 键: cgroup类别名 (例如 `TopApp`, `Background`)。这些键名应与程序内部（`CgroupManager::cgroup_categories_`）定义的类别键以及 `[CgroupPaths]` 中使用的前缀部分相对应。
* 值: CPU核心列表字符串。**此字符串现在可以使用核心类型名称** (例如 "SmallCore", "MediumCore", "BigCore", "SuperBigCore")、特殊关键字 (例如 "SmallestSingle", "AllCores" - 如果 `CpuUtils` 支持这些关键字的解析) 以及传统的数字和数字范围 (例如 "0-3", "7", "0,1,4-6")。 [cite: 1]
    * 程序会使用 `CpuUtils::CpuListParser::resolve_symbolic_cpu_list` 函数，并结合从 `Scheduler` 获取的当前设备核心类型映射，来动态解析这些字符串为实际的CPU核心ID列表。
    * **容错性**：如果配置中指定的核心类型名称（如 "SuperBigCore"）是一个程序已知的标准核心类型关键字，但在当前设备上并未被 `Scheduler` 检测到（即核心映射中不存在该类型，或该类型对应的CPU ID列表为空），那么在解析该CPU列表字符串时，这个**无法映射的核心类型名称将被跳过**，不会导致整个列表的解析失败。解析器会继续尝试解析和使用字符串中的其他有效部分。
    * 只有当字符串中的某个段**既不是已知的核心类型关键字，也不是已知的特殊关键字，也无法被解析为有效的数字或数字范围**时，才可能导致整个列表字符串的解析失败。
    * 示例:
        ```ini
        TopApp = "MediumCore,BigCore,SuperBigCore" ; 如果设备没有SuperBigCore，则实际生效的是MediumCore和BigCore绑定的核心
        Background = "SmallCore"
        Restricted = "SmallestSingle" ; 通常会解析为SmallCore中的一个核心 (具体行为取决于CpuUtils的实现)
        ```

---

#### 10. `[AffinitySettings]` - CPU亲和性设置

为不同cgroup类别（或进一步结合特定模式）设置CPU亲和性，即将这些cgroup中的进程绑定到特定的CPU核心上。由 `AffinityManager::apply_affinity_settings` 读取。

* **格式**:
    * `Affinity_CgroupCategoryKey = "核心列表字符串"`: 为指定的cgroup类别（`CgroupCategoryKey`，如 `TopApp`, `Background`）设置一个通用的CPU亲和性。 [cite: 1]
    * `Affinity_CgroupCategoryKey_ModeName = "核心列表字符串"`: 为指定的cgroup类别在特定的调度模式下（`ModeName`，如 `powersave`, `performance`）设置CPU亲和性。此设置会覆盖上面更通用的设置。 [cite: 1]
* **值**: CPU核心列表字符串，其格式和解析逻辑与 `[CpusetSettings]` 中的完全相同（同样支持核心类型名称、特殊关键字、数字和范围，并具有相同的容错跳过逻辑）。 [cite: 1]
* **优先级**: 当决定某个cgroup在某个模式下的亲和性时，查找顺序通常是：`app_profiles.ini` 中针对当前应用的特定亲和性配置 > `[AffinitySettings]` 中针对此cgroup和此模式的特定配置 (`Affinity_Xxx_ModeName`) > `[AffinitySettings]` 中针对此cgroup的通用配置 (`Affinity_Xxx`)。
    * 示例:
        ```ini
        Affinity_TopApp = "MediumCore,BigCore,SuperBigCore"
        Affinity_Background = "SmallCore"
        Affinity_Background_powersave = "SmallestSingle" ; 在powersave模式下，后台任务的亲和性进一步限制
        ```

---

#### 11. `[AppLaunchBoostSettings]` - 应用启动加速设置

控制应用启动时临时提升性能以加快加载速度的功能。

* **`Enable`**: 布尔值 (`true` 或 `false`)。是否启用应用启动加速功能。 [cite: 1]
* **`BoostDurationMs`**: 整数。应用启动加速效果的持续时间，单位为毫秒。在此期间，服务会应用下面指定的 `BoostModeName`。 [cite: 1]
* **`BoostModeName`**: 字符串。在应用启动加速期间应用的调度模式的名称。这个名称必须对应在本配置文件后面“模式定义”部分中定义的一个模式节名（例如 `performance` 或一个专门为启动加速设计的模式）。 [cite: 1]
* **`CoolDownMs`**: 整数。一次启动加速效果结束后，到下一次可以触发应用启动加速的冷却时间（或称为“不应期”），单位为毫秒。设置为 `0` 表示没有冷却时间，每次符合条件的应用启动都会触发加速。 [cite: 1]
* **`ExcludedApps`**: 字符串，一个**逗号分隔**的应用包名列表。当这些列表中的应用启动到前台时，**不会**触发启动加速功能。 [cite: 1]
    * 示例:
        ```ini
        Enable = true
        BoostDurationMs = 2500
        BoostModeName = performance
        CoolDownMs = 4000
        ExcludedApps = "com.android.launcher,com.android.systemui,com.android.settings"
        ```

---

#### 12. 模式定义 (例如 `[powersave]`, `[balance]`, `[performance]`, `[fast]`)

这些是最核心的配置节，定义了不同工作场景下的系统性能和功耗表现。每个模式都是一个独立的节，节名即为模式名（例如 `[balance]`）。`Scheduler::apply_mode` 函数会读取并应用当前激活模式下的这些参数。

**在一个模式节内 (以 `[balance]` 为例，您可以定义多个这样的模式节):**

* **全局调速器 (可选，对此模式生效)**:
    * `scaling_governor = schedutil`
        * 如果定义此项，则该模式下所有未在下面单独指定调速器的CPU核心簇都将使用这里设置的调速器。
        * 如果不定义此项，则每个核心簇必须单独指定其调速器，或者程序可能会使用一个内置的默认值（例如 `schedutil`）。

* **针对特定核心簇的参数**:
    * 这里的 `CoreType` 应替换为您在 `[CoreFramework]` 中定义或程序自动识别出的核心类型名称，例如 `SmallCore`, `MediumCore`, `BigCore`, `SuperBigCore`。
    * **调速器**: `CoreTypeScalingGovernor = governor_name`
        * 为指定的核心簇设置CPU调速器。会覆盖上面定义的模式全局 `scaling_governor`。
        * 示例: `SmallCoreScalingGovernor = schedutil`
    * **最大频率**: `CoreTypeMaxFreq = frequency_in_kHz | max`
        * 设置该核心簇的CPU最大运行频率，单位kHz。
        * `max` 是一个特殊值，表示允许该核心簇运行到其硬件支持的最高频率（从 `cpuinfo_max_freq` 读取）。
        * 示例: `BigCoreMaxFreq = max`, `SmallCoreMaxFreq = 1804800`
    * **最小频率**: `CoreTypeMinFreq = frequency_in_kHz | min`
        * 设置该核心簇的CPU最小运行频率，单位kHz。
        * `min` 是一个特殊值，表示允许该核心簇降低到其硬件支持的最低频率（从 `cpuinfo_min_freq` 读取）。
        * 示例: `SmallCoreMinFreq = 600000`
    * **Schedutil 特定参数 (仅当该核心簇的调速器被有效设置为 `schedutil` 时，这些参数才会尝试被应用)**:
        * `CoreTypeSchedutilUpRateLimitUs = value_in_microseconds`
        * `CoreTypeSchedutilDownRateLimitUs = value_in_microseconds`
            * **重要提示**: 如果您的设备内核在对应CPU的 `/sys/.../cpufreq/schedutil/` 目录下只有一个名为 `rate_limit_us` 的文件（而不是独立的 `up_rate_limit_us` 和 `down_rate_limit_us` 文件），那么程序会尝试将 `CoreTypeSchedutilUpRateLimitUs` 配置的值写入这个单一的 `rate_limit_us` 文件。在这种情况下，`CoreTypeSchedutilDownRateLimitUs` 的配置值会被忽略。请根据您设备内核的实际情况，将期望的 `rate_limit_us` 值填入对应的 `CoreTypeSchedutilUpRateLimitUs`。
        * 其他可能的 `schedutil` 参数，通常程序会将配置键中的驼峰命名（例如 `HispeedLoad`）转换为内核节点期望的下划线命名（例如 `hispeed_load`）并尝试写入。例如：
            * `CoreTypeSchedutilHispeedLoad = value` (0-100)
            * `CoreTypeSchedutilHispeedFreq = frequency_in_kHz`
            * `CoreTypeSchedutilTargetLoads = "comma,separated,values"` (如果节点接受此类格式)
            * `CoreTypeSchedutilAboveHispeedDelay = value_in_us_or_other_format`
        * 示例:
            ```ini
            SmallCoreSchedutilUpRateLimitUs = 8000
            SmallCoreSchedutilDownRateLimitUs = 16000
            BigCoreSchedutilHispeedLoad = 90
            ```

* **模式特定的 Uclamp 设置 (可选，会覆盖 `[UclampSettings_Default]` 中的同名项)**:
    * 这些参数通常针对特定的 cgroup 类别（如 `TopApp`, `Foreground`, `Background`）在该模式下的表现进行微调。
    * 键名中的 `CgroupCategoryKey` 应替换为实际的cgroup类别键，例如 `TopApp`, `Foreground`。
    * `UclampMin_CgroupCategoryKey = value` (通常是 0-1024 之间的整数)
    * `UclampMax_CgroupCategoryKey = value` (通常是 0-1024 之间的整数, 且应 >= UclampMin)
    * `UclampLatencySensitive_CgroupCategoryKey = 0_or_1` (0 表示禁用，1 表示启用延迟敏感提示)
    * 示例:
        ```ini
        UclampMin_TopApp = 20 
        UclampMax_TopApp = 90
        UclampLatencySensitive_TopApp = 1 ; 在性能模式下为TopApp开启延迟敏感
        ```

* **模式特定的 Cpuset 设置 (可选，会覆盖 `[CpusetSettings]` 中的同名项)**:
    * 如果希望某个模式对特定的 cgroup 类别使用与 `[CpusetSettings]` 中不同的CPU核心绑定。
    * `Cpuset_CgroupCategoryKey = "核心列表字符串"` (核心列表字符串的格式和解析逻辑同 `[CpusetSettings]`)
    * 示例: `Cpuset_Background = "SmallCore0"` (在某个非常省电的模式下，将后台任务严格限制到小核中的第一个核心)

* **直接写入 Sysfs 节点 (高级功能)**:
    * `SysfsPath_/path/to/sysfs/node = value_to_write`
    * 允许您在此模式被激活时，向任意指定的 sysfs 文件节点写入一个值。
    * `/path/to/sysfs/node` **必须是绝对路径**。
    * 此功能提供了极大的灵活性，但也要求用户非常清楚所操作节点的具体含义和允许的值，错误的操作可能导致系统不稳定。
    * 示例: `SysfsPath_/sys/module/some_kernel_module/parameters/some_tunable = 1`

---

## 🤝 贡献 (Contributing)

欢迎通过 Pull Requests 贡献代码，或在 Issues 中报告问题和提出建议。

## 📜 许可证 (License)

本项目采用 MIT 许可证。
