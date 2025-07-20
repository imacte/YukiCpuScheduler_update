# 🚀 YukiCpuScheduler v2.2.0 更新日志

> 💡 **小贴士**: 模块会自动为你匹配最佳的性能配置文件。想获得极致体验？可以随时在 `/data/adb/modules/YukiCpuScheduler/configs/` 目录下手动调整 `config.yaml` 和 `thread.yaml`。

## 📋 更新详情

### ✨ **新功能 (New Features)**

1.  **引入动态调速器参数引擎 (`Govsets`)**
    *   现在，你可以在 `config.yaml` 中通过全新的 `pGovPath` 和 `Govsets` 选项，对 CPU 调速器的内部参数（如 `up_rate_limit_us`, `target_loads` 等）进行精细化调整。
    *   **智能路径拼接**：程序会根据当前核心使用的调速器（`schedutil`, `walt` 等），自动构建正确的参数文件路径。
    *   **模式化配置**：可以为每种性能模式（省电、均衡、性能等）设置一套完全不同的调速器参数，实现前所未有的自定义深度。
    *   这项新功能让你能够真正挖掘出内核的全部潜力，打造最适合你使用习惯的性能曲线！

### 🛠️ **核心重构 (Refactoring)**

1.  **线程亲和性管理功能解耦**
    *   原有的、通过 `thread.yaml` 控制的特定线程核心绑定功能，已被**完全分离**，未来可能将作为一个独立的、可选的模块提供。
    *   本次更新后，YukiCpuScheduler 将更专注于 CPU 核心调度、频率和调速器参数的宏观管理。
    *   这使得主模块更加轻量、稳定，并为更专业的线程级调度模块提供了共存空间。

## 🔗 相关链接

  - **GitHub 仓库**: [https://github.com/imacte/YukiCpuScheduler\_update](https://github.com/imacte/YukiCpuScheduler_update)
  - **Gitee 仓库**: [https://gitee.com/imacte\_ui/YukiCpuScheduler\_update](https://gitee.com/imacte_ui/YukiCpuScheduler_update)
  - **问题反馈**: [GitHub Issues](https://github.com/imacte/YukiCpuScheduler_update/issues)
