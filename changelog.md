# 🚀 YukiCpuScheduler v2.0.9 更新日志

> 💡 **小贴士**: 模块会自动为你匹配最佳的性能配置文件。想获得极致体验？可以随时在 `/data/adb/modules/YukiCpuScheduler/configs/` 目录下手动调整 `config.yaml` 和 `thread.yaml`。

## 📋 更新详情

### ✨ 核心功能更新亮点

1.  **🚀 闪电般的模式切换**
    * **瞬时响应**：通过 `config.txt` 文件切换性能模式（例如，从“均衡”切换到“性能”）现在几乎是即时的。
    * **技术优化**：移除了不必要的磁盘读取和解析过程。所有性能模式的配置都已预先加载到内存中，使切换速度提升数十倍。

2.  **🧠 智能高效的配置热重载**
    * **精准监控**：重写了配置文件监控逻辑，系统现在能够独立且高效地监控 `config.yaml` 和 `config.txt` 的变化。
    * **智能应用**：
        * 修改 `config.yaml`（核心配置文件）会触发智能的 **完整重载**。
        * 修改 `config.txt`（模式切换文件）仅会进行轻量级的 **模式应用**，避免了不必要的系统开销。

3.  **🏗️ 更健壮的底层架构**
    * **统一管理**：所有性能模式的配置数据现在被统一存储和管理。
    * **面向未来**：这不仅让代码更清晰、更稳定，也为未来开发更复杂的新功能（如场景化调度、自定义模式等）奠定了坚实的基础。

4.  **🛠️ 全面的代码现代化**
    * **质量提升**：运用现代C++的最佳实践重构了代码，显著提升了代码质量。
    * **安全可靠**：确保了在多线程环境下的内存安全和线程安全，让服务运行更加稳定、安心。

5.  **🛠️ 新增*
    * **游戏帧率调谐器 (FAS)**：测试中。

### 🔧 修复 (Fixed)

  - **[性能]** 


## 🔗 相关链接

  - **GitHub 仓库**: [https://github.com/imacte/YukiCpuScheduler\_update](https://github.com/imacte/YukiCpuScheduler_update)
  - **Gitee 仓库**: [https://gitee.com/imacte\_ui/YukiCpuScheduler\_update](https://gitee.com/imacte_ui/YukiCpuScheduler_update)
  - **问题反馈**: [GitHub Issues](https://github.com/imacte/YukiCpuScheduler_update/issues)