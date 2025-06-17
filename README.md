# YukiCpuScheduler
[![C++](https://img.shields.io/badge/language-C++-%23f34b7d.svg?style=plastic)](https://en.wikipedia.org/wiki/C++)
[![Android](https://img.shields.io/badge/platform-Android-0078d7.svg?style=plastic)](https://en.wikipedia.org/wiki/Android_(operating_system)) 
[![AArch64](https://img.shields.io/badge/arch-AArch64-red.svg?style=plastic)](https://en.wikipedia.org/wiki/AArch64)
[![Android Support8-15](https://img.shields.io/badge/Android%208~15-Support-green)](https://img.shields.io/badge/Android%208~15-Support-green)
#### 介绍
YukiCpuScheduler是一款基于 C++ 编写的智能 CPU 调度工具 旨在优化 Android 设备的 CPU 性能和功耗表现而设计 通过智能调度算法 它可以根据不同的使用场景动态调整 CPU 频率以达到最佳的性能和能效平衡 <br>
#### 工作条件
1.目前该调度适用于ARM64平台的Android8-15 <br>
2.拥有Root权限

### 情景模式
- powersave 省电
- balance 均衡
- performance 性能
- fast 极速

#### 日志等级
- DEBUG 调试
- INFO 信息
- WARNNING 警告
- ERROR 错误
  
#### 情景模式切换
powersave（省电模式）：在保证基本流畅度的同时，尽可能降低功耗。推荐'待机'使用 <br>
balance（均衡模式）：比原厂设置更流畅且更省电。推荐'日常'使用 <br>
performance（性能模式）：在保证较高流畅度的同时，可能会增加功耗。推荐'需要短时间性能的场景'使用 <br>
fast（极速模式）：全力保证游戏时的流畅度，忽略能效比，并没有任何'频率限制'并且所有核心将拉升至最高频率

## 常见问题
Q：是否会对待机功耗产生负面影响？ <br>
A：YukiCpuScheduler 做了低功耗优化 由于使用了 C++ 语言 自身运行功耗很低 并不会对设备的待机功耗产生显著影响   <br>
 
Q：为什么使用了 YukiCpuScheduler 后功耗仍然很高？  <br> 
A：SOC 的 AP 部分功耗主要取决于计算量和使用的频点 CPU Turbo Scheduler 只能通过控制性能释放和改进频率的方式来降低功耗 如果后台应用的计算量很大 可能无法显著延长续航时间 可以通过 Scene 工具箱的进程管理器来定位问题  PS:建议全局测试时间不低于一小时 否则属于瞬时功耗 瞬时功耗不具有任何参考价值 <br>

Q：如何确保我的设备拥有并支持 Feas特性？  <br>
A：开启YukiCpuScheduler 的 Feas 开关并切换到极速模式YukiCpuScheduler将会自动识别内核的 Feas 接口 如果设备没有 Feas 功能接口 将会在日志中抛出错误 目前YukiCpuScheduler 已接入大多数内核的 Feas 接口  <br>

Q：是否还需要关闭系统的performance boost？  <br>
A：YukiCpuScheduler在初始化阶段就已经关闭了大部分主流的用户态和内核态升频 如果有非常规的升频需要用户自己关闭   <br>

Q：YukiCpuScheduler和Scene工具箱是什么关系？  <br>
A：这两个软件独立运作没有互相依赖 CPU Turbo Scheduler实现了接口可供Scene工具箱调用 例如性能模式切换以及分APP性能模式 如果不安装Scene工具箱也可以实现性能模式切换 <br>

Q：RubbishProcess指的是什么进程？ <br>
A：进程列表:'kswapd''logd''kcompactd''magiskd''zygiskd''init'为防止这些进程占用过高的CPU导致异常耗电 所以默认将这些进程绑定到0-2核心  <br>

Q：AffintySetter功能是否跟XX系统流畅度提升模块冲突  <br>
A：冲突 目前CPU Turbo Scheduler会对一些系统关键进程进行绑核操作 所以不必使用此类模块  <br>

Q：支不支持XXX内核？<br>
A：目前YukiCpuScheduler支持大部分内核 举例4.4及以上的内核 <br> 

Q：与'潘多拉''魔理沙'内核是否冲突？<br>
A：目前YukiCpuScheduler仅在刷入'潘多拉'内核的设备上测试过 可以使用但可能会导致某些核心的最大频率异常 PS:CS调度并未适配这些内核提供的Feas接口 <br> 

Q：切换情景模式后是否需要重启？<br>
A：不需要 目前YukiCpuScheduler会监听情景模式的配置变化进行切换 <br> 

Q：应用启动加速的频率是多少？<br>
A：应用启动加速会对不同的核心写入'SmallCoreMaxFreq' 'MediumCoreMaxFreq' 'BigCoreMaxFreq' 'SuperBigCoreMaxFreq'的1.25倍 PS:举例冷启动时将小核的频率拉升至'SmallCoreMaxFreq' 以此类推

## 配置文件说明
### （一）元信息（meta）

```ini
[meta]
name = "YukiCpuScheduler正式版模型"
author = MoWei
configVersion = 13
loglevel = "INFO"
```
| 字段名   | 数据类型 | 描述                                           |
| -------- | -------- | ---------------------------------------------- |
| name     | string   | 配置文件的名称                                 |
| author   | string   | 配置文件的作者信息                             |
| configVersion | string   | 配置文件的版本号 |
| loglevel | string   | 日志等级，可选值为 DEBUG、INFO、WARNING、ERROR |

### (二) 附加功能 （function）
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
| 字段名   | 数据类型 | 描述                                           |
| -------- | -------- | ---------------------------------------------- |
| DisableQcomGpu | bool   | 禁用高通 GPU Boost 防止 GPU 频率无序升高 | 
| AffintySetter | bool   | 对系统和传感器关键进程和线程进行绑核操作 |
| CpuIdleScaling_Governor | bool   | 自定义 CPUIdle 调度器。 |
| EasScheduler | bool   | EAS 调度器参数优化 |
| cpuset | bool | CPUSet功能 调整应用的核心分配 |
| LoadBalancing | bool | 通过优化CFS调度器的参数达到负载均衡的效果 |
| EnableFeas | bool |  FEAS 功能（仅限极速模式）|
| AdjIOScheduler | bool |  I/O 调度器调整以及I/O优化总开关 |
| AppLaunchBoost | bool |  APP启动加速 PS:在启动应用时对CPU0-7核心进行时长为1.2S的升频 无论是热启动或冷启动 频率可看上面的QA |

### （三）核心分配参数 （CoreAllocation）
```ini
[CoreAllocation]
cpusetCore = "4-7"
cpuctlUclampBoostMin = "0"
cpuctlUclampBoostMax = "100"
```
| 字段名   | 数据类型 | 描述                                           |
| -------- | -------- | ---------------------------------------------- |
| cpusetCore | string   | 指定 CPUSet核心 用于核心分配 |
| cpuctlUclampBoostMin | string   | CPU使用率控制的最小值 （范围0-100） |
| cpuctlUclampBoostMax | string   | CPU使用率控制的最大值 （范围0-100） |

### （四）核心架构参数 （CoreFramework）
##### 如果觉得这部分比较复杂 可以看上面的QA
```ini
[CoreFramework]
SmallCorePath = 0
MediumCorePath = 4
BigCorePath = 0
SuperBigCorePath = 0
```
| 字段名   | 数据类型 | 描述                                           |
| -------- | -------- | ---------------------------------------------- |
| SmallCorePath | int   | 代指小核的CPU路径 |
| MediumCorePath | int   | 代指中核的CPU路径 |
| BigCorePath | int   | 代指大核的CPU路径 |
| SuperBigCorePath | int   | 代指超大核的CPU路径 |

### （五）I/O 设置（IO_Settings）
```ini
[IO_Settings]
Scheduler = ""
IO_optimization = false
```
| 字段名   | 数据类型 | 描述                                           |
| -------- | -------- | ---------------------------------------------- |
| Scheduler | string   | 指定 I/O 调度器类型，如 ssg、bfq 等 PS:该值为空时 将不会修改I/O调度器 |
| IO_optimization | bool   | 启用 I/O 优化功能 |

### （六）QcomBus 参数优化（Other）
```ini
[Other]
AdjQcomBus_dcvs = false
```
| 字段名   | 数据类型 | 描述                                           |
| -------- | -------- | ---------------------------------------------- |
| AdjQcomBus_dcvs | bool   | 优化QCOM设备'DDR''LLCC''DDRQOS''L3'参数 PS:目前该功能在7GEN2+设备上使用最佳 |

### （七）EAS 调度器参数（EasSchedulerVaule）
```ini
[EasSchedulerVaule]
sched_min_granularity_ns = "2000000" 
sched_nr_migrate = "30"
sched_wakeup_granularity_ns = "3200000"
sched_schedstats = "0"
```
| 字段名   | 数据类型 | 描述                                           |
| -------- | -------- | ---------------------------------------------- |
| sched_min_granularity_ns | string   | EAS 调度器的最小调度粒度 单位为纳秒（ns） |
| sched_nr_migrate | string   | 控制任务在 CPU 核心之间迁移的次数 |
| sched_wakeup_granularity_ns | string   | EAS 调度器调整任务唤醒时间的粒度 单位为纳秒（ns） |
| sched_schedstats | string   | 是否启用调度统计信息收集(0表示禁用) |

### （八）CpuIdle 调度器（CpuIdle）
```ini
[CpuIdle]
current_governor = ""
```
| 字段名   | 数据类型 | 描述                                           |
| -------- | -------- | ---------------------------------------------- |
| current_governor | string   | 设置使用的 CpuIdle 调度器模式 如高通推荐:qcom-cpu-lpm 联发科推荐:menu PS:此值为空等于不调整 |

### （九）CPUSet 配置（Cpuset）
```ini
[Cpuset]
top_app = "0-7"
foreground = "0-7"
restricted = "0-5"
system_background = "1-2"
background = "0-2"
```
| 字段名   | 数据类型 | 描述                                           |
| -------- | -------- | ---------------------------------------------- |
| top_app | string   | 顶层应用可使用的 CPU 核心范围 |
| foreground | string   | 前台应用可使用的 CPU 核心范围 |
| restricted | string   | 前台任务加速时可使用的 CPU 核心范围 |
| system_background | string   | 系统后台进程可使用的 CPU 核心范围 |
| backgroundd | string   | 后台进程可使用的 CPU 核心范围 |

###  (十)功耗模型开发 (这里使用性能模式举例)
```ini
[performance]
scaling_governor = "schedutil"
UclampTopAppMin = "0"
UclampTopAppMax = "100"
UclampTopApplatency_sensitive = "1"
UclampForeGroundMin = "0"
UclampForeGroundMax = "80"
UclampBackGroundMin = "0"
UclampBackGroundMax =  "50"
SmallCoreMaxFreq = 10000
MediumCoreMaxFreq = 2500
BigCoreMaxFreq = 2700
SuperBigCoreMaxFreq = 2700
ufsClkGate = false
```
| 字段名   | 数据类型 | 描述                                           |
| -------- | -------- | ---------------------------------------------- |
| scaling_governor | string   | 指定0-7核心的CPU调速器 |
| UclampTopAppMin | string   | 用于设置顶层APP可使用的CPU频率下限 （范围0-100）|
| UclampTopAppMax | string   | 用于设置顶层APP可使用的CPU频率上限 （范围0-100）|
| UclampTopApplatency_sensitive | string   | 标记应用或进程对延迟敏感性的参数 设置此参数可以告知调度器前台应用对延迟非常敏感 需要优先处理以减少响应时间 PS:在老机型上此功能可能无法使用 |
| UclampForeGroundMin | string   | 用于设置前台APP可使用的CPU频率下限 （范围0-100）|
| UclampForeGroundMax | string   | 用于设置前台APP可使用的CPU频率上限 （范围0-100）|
| UclampBackGroundMin | string   | 用于设置后台APP可使用的CPU频率下限 （范围0-100）|
| UclampBackGroundMax | string   | 用于设置后台APP可使用的CPU频率上限 （范围0-100）|
| SmallCoreMaxFreq | int   | 用于设置小核的CPU最大频率 （范围0-10000） |
| MediumCoreMaxFreq | int   | 用于设置中核的CPU最大频率 （范围0-10000） |
| BigCoreMaxFreq | int   | 用于设置大核的CPU最大频率 （范围0-10000） |
| SuperBigCoreMaxFreq | int   | 用于设置超大核的CPU最大频率 （范围0-10000）|
| ufsClkGate | string   | 用于设置UFS时钟门 |

进阶功能：特定线程核心分配 (threads.ini)

除了全局的CPU调度策略外，本调度器还支持一项强大的进阶功能：为特定应用程序的特定线程绑定到指定的一个或一组CPU核心上运行。

这项功能对于游戏或专业应用中的关键线程（如渲染线程、UI主线程）特别有用，可以有效减少线程在不同核心间切换所带来的性能抖动，提升应用的稳定性。
配置文件

此功能的所有配置均在以下文件中完成：
/data/adb/modules/YukiCpuScheduler/configs/threads.ini
基本语法

配置文件采用标准的INI格式：

    [应用包名]: 使用中括号 [] 来定义一个配置段，括号内的内容是您想要管理的应用的完整包名。
    线程名 = 核心范围: 在每个应用配置段下，您可以添加多条规则。每条规则的格式为 键 = 值，其中“键”是线程的名称，“值”是您希望它运行的核心编号。

配置示例和功能说明

我们支持三种不同优先级的规则，让您可以进行非常灵活的配置。

1. 精确线程绑定 (最高优先级)
这是最基本的用法，将一个名字完全匹配的线程绑定到指定的核心。

    格式: 线程名 = 核心范围
    示例:
    Ini, TOML

    [com.tencent.tmgp.speedmobile]
    # 将名字精确为 "UnityMain" 的线程绑定到核心 6-7
    UnityMain = 6-7

2. 使用通配符 * 模糊匹配 (中等优先级)
这是新版本借鉴 apply.cpp 后增加的强大功能。您可以使用 * 作为通配符来匹配一类线程。

    格式: 线程*名*称* = 核心范围
    示例:
    Ini, TOML

    [com.tencent.tmgp.speedmobile]
    # 匹配所有以 "Render" 开头的线程 (如 RenderThread, RenderWorker01 等)
    # 将它们全部绑定到核心 7
    Render* = 7

    # 匹配所有名字中包含 "Audio" 的线程
    *Audio* = 0-1

3. 为应用设置“默认”核心组 (最低优先级)
这是一个非常实用的新功能。您可以用一个 * 作为键名来代表“此应用中所有其他未匹配到上述更具体规则的线程”。这相当于在 threads.ini 内部为这个特定应用设置了一个“默认值”。

    格式: * = 核心范围
    示例:
    Ini, TOML

    [com.tencent.tmgp.speedmobile]
    # 将《QQ飞车》中所有“普通”线程（即没匹配到下面两条规则的）
    # 全都限制在核心 2-5 上运行。
    * = 2-5

规则优先级

当一个线程同时可能匹配到多条规则时，调度器会按照从高到低的优先级来选择：

    精确名称匹配 (如 UnityMain)
    通配符名称匹配 (如 Render*)
    应用默认规则 (*)
    如果以上规则均未在threads.ini中设置，线程将被放置在系统默认的 top-app cpuset中。

完整配置范例

下面是一个综合运用了所有功能的 threads.ini 文件示例：
Ini, TOML

[meta]
name=Yuki App Profile Configuration
author=Yuki
version=2.0

# === QQ飞车配置 ===
[com.tencent.tmgp.speedmobile]

# 规则三 (优先级最低): 
# 首先，为这个游戏设定一个默认的核心范围 2-5。
# 所有线程一开始都会被假定要在这个范围运行。
* = 2-5

# 规则二 (优先级中): 
# 接着，为渲染相关的线程设定一个更具体的核心范围 7。
# 这会覆盖上面的默认规则。
Render* = 7

# 规则一 (优先级最高): 
# 最后，为最关键的 UnityMain 线程设定最精确的核心范围 6-7。
# 这会覆盖上面所有的规则。
UnityMain = 6-7
UnityGfxDeviceW = 5-7

如何查找应用的线程名？

您可以在应用运行时，通过 adb 执行以下命令来查看该应用的所有线程及其名称（请将 [PID] 替换为应用的实际进程ID）：
Shell
top -H -p [PID]

或者
Shell
ps -T -p [PID]

# 致谢 （排名不分前后）
感谢以下用户对本项目的帮助：  
- [Github@MoWei-2077](https://github.com/MoWei-2077) <br>
- [Github@ShenEternal](https://github.com/Asbzyyds) <br>
- [Github@AquaPuff🐋](https://github.com/zgquangui) <br>
- [CoolAPK@hfdem](https://github.com/hfdem) <br>
- [CoolAPK@ztc1997](https://github.com/ztc1997) <br>
- [CoolAPK@XShe](https://github.com/XShePlus) <br>
- [CoolAPK@Timeline](https://github.com/nep-Timeline) <br>
- [CoolAPK@shrairo](https://github.com/shrairo) <br>
- QQ@长虹久奕
- QQ@Microsoft
- QQ@:枫
- 各位酷友以及yuki调度的所有用户
# 使用的开源项目
- 暂无 <br>

### 该文档更新于:2025/04/18 19:46
- 感谢所有用户的测试反馈 这将推进YukiCpuScheduler的开发
