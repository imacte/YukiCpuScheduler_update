# 🚀 YukiCpuScheduler 版本更新

> ⚠️ **重要提示**  
> 更新配置文件不是看模块版本，而是看 [GitHub](https://github.com/imacte/YukiCpuScheduler_update) 或 [Gitee](https://gitee.com/imacte_ui/YukiCpuScheduler_update) 仓库 `configs` 和 `threads` 文件夹里的内容是否更新。如果更新，重新安装模块即可。

## 🔧 修复与优化

### 🛠️ 待机监测优化
- **🎯 修复天玑平台卡死问题**：解决开机使用省电模式的 `powersave` 调速器导致系统卡死的问题
- **⚙️ 调速器切换**：将默认调速器从 `powersave` 更改为更稳定的 `schedutil`，提升系统稳定性
- **📱 设备兼容性**：修复部分机型下载配置文件的问题
- **📥 下载优化**：优化了下载配置文件的流程和稳定性

## ✨ 新增功能

### 🎯 CPU 核心分配功能
- **🌐 备用下载地址**：新增 Gitee 备用下载地址，提升下载成功率
- **📝 手动配置支持**：全新支持通过 `threads.ini` 文件手动分配 CPU 核心
- **🔄 灵活调节**：支持根据不同场景自定义核心使用策略
- **🆕 线程规则扩展**：新增骁龙 8 Gen 2 的线程规则支持
- **📚 配置指南**：详细配置方法请参考 **GitHub 仓库**：https://github.com/imacte/YukiCpuScheduler_update

## 📱 支持平台

### 🔥 骁龙系列

| 处理器型号 | 支持状态 |
|------------|----------|
| 骁龙 870 | ✅ 完全支持 |
| 骁龙 888 | ✅ 完全支持 |
| 骁龙 8 Gen 1 | ✅ 完全支持 |
| 骁龙 8+ Gen 1 | ✅ 完全支持 |
| 骁龙 8 Gen 2 | ✅ 完全支持 |
| 骁龙 8 Gen 3 | ✅ 完全支持 |
| 骁龙 7+ Gen 2 | ✅ 完全支持 |

## 📖 使用说明

### 🎯 核心分配配置
想要了解如何配置 **CPU 核心分配功能**？

**👉 [访问 GitHub 仓库获取详细配置教程](https://github.com/imacte/YukiCpuScheduler_update)**

### 🔧 快速开始
1. 下载最新版本
2. 将文件解压到设备中
3. 根据需求配置 `threads.ini` 文件
4. 应用设置，享受优化后的性能表现

---

## 📋 更新摘要

- ✅ 新增 Gitee 备用下载地址
- ✅ 修复天玑平台系统卡死问题
- ✅ 提升调速器稳定性
- ✅ 新增手动核心分配支持
- ✅ 优化下载配置文件流程
- ✅ 新增骁龙 8 Gen 2 线程规则
- ✅ 修复配置文件下载问题
- ✅ 增强设备兼容性

---

**💡 专业提示**：为获得最佳性能表现，建议根据您的具体使用习惯和设备性能来自定义 `threads.ini` 配置文件。

---

*🌟 **让您的设备性能调度更智能，体验更流畅！** ✨*

## 🔗 相关链接

- **GitHub 仓库**：https://github.com/imacte/YukiCpuScheduler_update
- **Gitee 仓库**：https://gitee.com/imacte_ui/YukiCpuScheduler_update
- **问题反馈**：[Issues](https://github.com/imacte/YukiCpuScheduler_update/issues)
- **更新日志**：[Releases](https://github.com/imacte/YukiCpuScheduler_update/releases)