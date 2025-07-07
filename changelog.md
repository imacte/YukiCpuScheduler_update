# 🚀 YukiCpuScheduler v2.0.8 更新日志

> 💡 **小贴士**: 模块会自动为你匹配最佳的性能配置文件。想获得极致体验？可以随时在 `/data/adb/modules/YukiCpuScheduler/configs/` 目录下手动调整 `config.yaml` 和 `thread.yaml`。

-----

## ✨ 本次更新亮点

本次更新带来了重大的配置系统重构！现在支持为不同核心簇独立设置CPU调速器，实现了前所未有的精细化性能控制。应用启动加速逻辑也得到全面优化，采用独立的配置，确保每次启动都动力十足。同时，我们修复了一个导致CPU频率异常尖峰的BUG，使性能调度更加平稳高效。配置文件结构更清晰，欢迎体验全新的自定义能力！

-----

## 📋 更新详情

### ✨ 新增与增强 (Features & Enhancements)

  - **[核心]** 引入结构化的性能模式配置：`config.yaml` 中每个性能模式（如 `powersave`, `balance` 等）的配置项现在被归类到 `Governor` (调速器), `Freq` (频率), `Uclamp`, 和 `Other` (其他) 四个子块中。这使得配置文件结构更清晰，逻辑分组更合理，极大地提高了可读性和可维护性。
  - **[核心]** 支持为不同CPU核心簇独立设置调速器：现在可以在 `Governor` 配置块下，为 `SmallCore`, `MediumCore`, `BigCore`, `SuperBigCore` 单独指定调速器。同时保留了 `global` 关键字用于设置全局默认调速器，独立设置的优先级高于全局，提供了前所未有的灵活性。
  - **[配置]** 新增了对 **骁龙 8 至尊版** 的支持。

### 🚀 优化 (Optimized)

  - **[配置]** 配置加载逻辑重构：重写了配置加载函数，使其能够智能解析新的结构化配置，并向后兼容。优化了频率值的解析，现在可以直接在配置文件中使用 `"max"` 或 `"min"` 字符串来代表硬件支持的最高或最低频率，增强了通用性。
  - **[代码]** 代码健壮性提升：在应用调速器设置前，增加了对 `cpufreq/policyX` 路径是否存在且可写的检查，避免了在某些设备上因核心簇不存在而产生的错误日志。
  - **[安装]** 增加了在安装时选择是否禁用厂商性能服务（如小米Joyose）的选项，将控制权交给用户。

### 🔧 修复 (Fixed)

  - **[性能]** 修复了在应用配置，CPU频率会瞬间飙升至硬件最大值的BUG，频率设置现在更加平滑、直接。


-----

## 📱 支持的设备平台

我们为最新的主流芯片提供了开箱即用的优化配置。

<details>
<summary><b>🔥 点击展开骁龙 (Qualcomm Snapdragon) 支持列表</b></summary>

| 处理器型号 | 主配置文件 | 线程配置文件 |
|---|---|---|
| 骁龙 8 至尊版 | ✅ | ❌ (即将支持) |
| 骁龙 8 Gen 3 (sm8650) | ✅ | ✅ |
| 骁龙 8 Gen 2 (sm8550) | ✅ | ✅ |
| 骁龙 8+ Gen 1 (sm8475) | ✅ | ❌ (即将支持) |
| 骁龙 8 Gen 1 (sm8450) | ✅ | ❌ (即将支持) |
| 骁龙 888 (sm8350) | ✅ | ❌ (即将支持) |
| 骁龙 870 (sm8250-ac) | ✅ | ❌ (即将支持) |
| 骁龙 855 (sm8150) | ✅ | ❌ (即将支持) |
| 骁龙 7+ Gen 2 (sm7475) | ✅ | ❌ (即将支持) |

</details>

<details>
<summary><b>💎 点击展开天玑 (MediaTek Dimensity) 支持列表</b></summary>

| 处理器型号 | 主配置文件 | 线程配置文件 |
|---|---|---|
| 天玑 9300 (mt6989) | ✅ | ❌ (即将支持) |
| 天玑 9200+ (mt6985z) | ✅ | ❌ (即将支持) |
| 天玑 9200 (mt6985) | ✅ | ❌ (即将支持) |
| 天玑 9000+ (mt6983z) | ✅ | ❌ (即将支持) |
| 天玑 9000 (mt6983) | ✅ | ❌ (即将支持) |
| 天玑 8100 (mt6895) | ✅ | ❌ (即将支持) |
| 天玑 6080 (mt6080) | ✅ | ❌ (即将支持) |

</details>

> `❌` 表示该芯片暂未提供专属的 `thread.yaml` 文件，但你依然可以自己创建并使用该功能！

-----

## 📖 快速上手

1.  在 Magisk Manager 中安装最新版本的模块。
2.  在安装过程中，根据提示使用**音量键**选择你的下载源和云控策略。
3.  重启手机，模块将自动生效。
4.  想要极致自定义？请编辑 `/data/adb/modules/YukiCpuScheduler/configs/` 目录下的 `.yaml` 文件。

**👉 [点击这里，查看详细的 `thread.yaml` 配置教程](https://www.google.com/search?q=%5Bhttps://github.com/imacte/YukiCpuScheduler_update%5D\(https://github.com/imacte/YukiCpuScheduler_update\))**

-----

*🌟 **让你的设备性能，由你掌控！** ✨*

## 🔗 相关链接

  - **GitHub 仓库**: [https://github.com/imacte/YukiCpuScheduler\_update](https://github.com/imacte/YukiCpuScheduler_update)
  - **Gitee 仓库**: [https://gitee.com/imacte\_ui/YukiCpuScheduler\_update](https://gitee.com/imacte_ui/YukiCpuScheduler_update)
  - **问题反馈**: [GitHub Issues](https://github.com/imacte/YukiCpuScheduler_update/issues)