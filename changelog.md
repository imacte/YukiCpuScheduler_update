# 🚀 YukiCpuScheduler 版本更新

> ⚠️ **重要提示** > 更新配置文件不是看模块版本，而是看 [GitHub](https://github.com/imacte/YukiCpuScheduler_update) 或 [Gitee](https://gitee.com/imacte_ui/YukiCpuScheduler_update) 仓库 `configs` 和 `threads` 文件夹里的内容是否更新。如果更新，重新安装模块即可。

## 🔧 修复与优化

### ⚙️ 核心优化 (Core Optimizations)
- **🚀 配置文件格式升级**：项目核心配置文件已从 `.ini` 格式全面升级为 `.yaml` 格式，带来了更强的结构化能力和可读性，便于用户理解和修改。
- **🧵 线程调度代码重构**：完全重构了线程亲和性管理模块的代码，使其逻辑更清晰，性能更稳定，为未来的功能扩展打下坚实基础。
- **📥 下载流程优化**：优化了配置文件的下载流程，提升了下载过程的稳定性和成功率。

### 📱 兼容性修复
- **修复部分机型下载配置文件的问题**
- **增强了对不同 Android 版本的设备兼容性**
- - **修复配置文件读取长度问题**

## ✨ 新增功能

### 🎯 CPU 核心分配与控制
- **开关式线程亲和性**：现在可以在 `config.yaml` 文件中通过 `EnableThreadAffinity: true/false` 选项，自由开启或关闭“特定线程核心分配”功能，默认开启。
- **手动配置支持**：全新的 `threads.yaml` 文件允许用户为特定应用的特定线程手动分配 CPU 核心。
- **灵活调节**：支持根据不同场景和应用，自定义精细化的核心使用策略。

## 📱 支持平台

### 🔥 骁龙系列 (Qualcomm Snapdragon)

| 处理器型号 | 主配置文件 (`-config.yaml`) | 线程配置文件 (`-thread.yaml`) |
|---|---|---|
| 骁龙 8 Gen 3 (sm8650) | ✅ | ✅ |
| 骁龙 8 Gen 2 (sm8550) | ✅ | ✅ |
| 骁龙 8+ Gen 1 (sm8475) | ✅ | ❌ |
| 骁龙 8 Gen 1 (sm8450) | ✅ | ❌ |
| 骁龙 888 (sm8350) | ✅ | ❌ |
| 骁龙 870 (sm8250-ac) | ✅ | ❌ |
| 骁龙 855 (sm8150) | ✅ | ❌ |
| 骁龙 7+ Gen 2 (sm7475) | ✅ | ❌ |

### 💎 天玑系列 (MediaTek Dimensity)

| 处理器型号 | 主配置文件 (`-config.yaml`) | 线程配置文件 (`-thread.yaml`) |
|---|---|---|
| 天玑 6080 (mt6080) | ✅ | ❌ |
| 天玑 8100 (mt6895) | ✅ | ❌ |
| 天玑 9200 (mt6985) | ✅ | ❌ |
| 天玑 9200+ (mt6985z) | ✅ | ❌ |
| 天玑 9300 (mt6989) | ✅ | ❌ |

## 📖 使用说明

### 🎯 核心分配配置
想要了解如何配置 **CPU 核心分配功能**？

**👉 [访问 GitHub 仓库获取详细配置教程](https://github.com/imacte/YukiCpuScheduler_update)**

### 🔧 快速开始
1. 下载最新版本的 Magisk 模块。
2. 在 Magisk Manager 中安装。
3. （可选）根据需求修改 `/data/adb/modules/YukiCpuScheduler/configs/` 目录下的 `config.yaml` 和 `threads.yaml` 文件。
4. 重启手机，享受优化后的性能表现。

---

## 📋 更新摘要

- ✅ **新增** 天玑6080、天玑9200+、骁龙855的配置文件
- ✅ **新增** `threads.yaml` 手动核心分配支持
- ✅ **新增** `EnableThreadAffinity` 功能开关
- ✅ **优化** 配置文件全面升级为 YAML 格式
- ✅ **优化** 重构线程调度代码
- ✅ **优化** 提升了调速器和下载流程的稳定性
- ✅ **修复** 配置文件下载问题
- ✅ **修复** 修复配置文件读取长度问题

---

**💡 专业提示**：为获得最佳性能表现，建议根据您的具体使用习惯和设备性能，来自定义 `config.yaml` 和 `threads.yaml` 配置文件。

---

*🌟 **让您的设备性能调度更智能，体验更流畅！** ✨*

## 🔗 相关链接

- **GitHub 仓库**：https://github.com/imacte/YukiCpuScheduler_update
- **Gitee 仓库**：https://gitee.com/imacte_ui/YukiCpuScheduler_update
- **问题反馈**：[Issues](https://github.com/imacte/YukiCpuScheduler_update/issues)
- **更新日志**：[Releases](https://github.com/imacte/YukiCpuScheduler_update/releases)
