# 🚀 YukiCpuScheduler v2.0.7 更新日志

> 💡 **小贴士**: 模块会自动为你匹配最佳的性能配置文件。想获得极致体验？可以随时在 `/data/adb/modules/YukiCpuScheduler/configs/` 目录下手动调整 `config.yaml` 和 `thread.yaml`。

---

## ✨ 本次更新亮点

### ✨ 更新简要
- **[代码]** 彻底修复最小频率设置问题。


### 🎯 **全新线程亲和性引擎 (New Thread Affinity Engine)**
- **精准核心绑定**: 新增了强大的 `thread.yaml` 配置文件，现在你可以为特定应用中的特定线程（如游戏的渲染线程、工作线程）精确绑定到指定CPU核心！这能显著减少卡顿，提升高负载下的流畅度。
- **完全可控**: 在 `config.yaml` 中新增 `EnableThreadAffinity` 开关，让你一键启用或禁用此功能，在极致性能和默认调度之间自由切换。
- **代码完全重构**: 底层线程管理代码被完全重写，带来了前所未有的稳定性和响应速度，并为未来的高级功能（如动态绑定）奠定了基础。

### ⚙️ **配置系统全面升级 (Total Configuration Revamp)**
- **拥抱 YAML**: 所有 `.ini` 配置文件已光荣退役，全面升级为更强大、更直观的 `.yaml` 格式。编辑和理解配置从未如此简单。
- **智能在线匹配**: 优化了安装时的在线配置文件匹配逻辑，现在能更智能、更快速地为你的设备匹配最佳的初始设置。修复了部分天玑（MediaTek）和特殊设备型号无法下载配置的问题。

---

## 📋 更新详情

### ✨ 新增 (Added)
- **[核心]** 新增 `thread.yaml` 文件，支持手动配置线程到核心的亲和性规则。
- **[核心]** 在 `config.yaml` 中新增 `EnableThreadAffinity` 开关，用于控制线程绑定功能。
- **[配置]** 新增了对天玑 9000/9000+/9200+/6080 和骁龙 855 的官方支持。

### 🚀 优化 (Optimized)
- **[架构]** `config.ini` 全面升级为 `config.yaml`，提升可读性和扩展性。
- **[性能]** 完全重构线程调度核心代码，运行更高效、更稳定。
- **[网络]** 优化了安装脚本中的配置文件下载流程，提高了成功率和速度。
- **[安装]** 增加了在安装时选择是否禁用厂商性能服务（如小米Joyose）的选项，将控制权交给用户。

### 🔧 修复 (Fixed)
- **[代码]** 彻底修复最小频率设置问题。
- **[兼容性]** 彻底修复了部分天玑(MediaTek)设备因型号识别问题而无法下载配置文件的bug。
- **[兼容性]** 修复了因配置文件过长可能导致的读取不完整问题。
- **[安装]** 修复了在某些设备上按一次音量键会触发两次选择的bug。

---

## 📱 支持的设备平台

我们为最新的主流芯片提供了开箱即用的优化配置。

<details>
<summary><b>🔥 点击展开骁龙 (Qualcomm Snapdragon) 支持列表</b></summary>

| 处理器型号 | 主配置文件 | 线程配置文件 |
|---|---|---|
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

---

## 📖 快速上手

1.  在 Magisk Manager 中安装最新版本的模块。
2.  在安装过程中，根据提示使用**音量键**选择你的下载源和云控策略。
3.  重启手机，模块将自动生效。
4.  想要极致自定义？请编辑 `/data/adb/modules/YukiCpuScheduler/configs/` 目录下的 `.yaml` 文件。

**👉 [点击这里，查看详细的 `thread.yaml` 配置教程](https://github.com/imacte/YukiCpuScheduler_update)**

---

*🌟 **让你的设备性能，由你掌控！** ✨*

## 🔗 相关链接

- **GitHub 仓库**: [https://github.com/imacte/YukiCpuScheduler_update](https://github.com/imacte/YukiCpuScheduler_update)
- **Gitee 仓库**: [https://gitee.com/imacte_ui/YukiCpuScheduler_update](https://gitee.com/imacte_ui/YukiCpuScheduler_update)
- **问题反馈**: [GitHub Issues](https://github.com/imacte/YukiCpuScheduler_update/issues)