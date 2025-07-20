# 🚀 YukiCpuScheduler v2.1.0 更新日志

> 💡 **小贴士**: 模块会自动为你匹配最佳的性能配置文件。想获得极致体验？可以随时在 `/data/adb/modules/YukiCpuScheduler/configs/` 目录下手动调整 `config.yaml` 和 `thread.yaml`。

## 📋 更新详情

### ✨ 新增 (Features)

-   **[核心功能]** 新增了高度可配置的**总线频率控制 (`Bus_dcvs`)** 功能！
    -   现在，您可以为每个性能模式独立设置 **LLCC 缓存**和 **DDR 内存**的频率范围，显著降低高负载下的数据访问延迟，全面提升系统响应速度和流畅度。
    -   该功能通过智能路径检测，**同时兼容高通 (Qualcomm) 的 `min/max_freq` 和天玑 (MediaTek) 的 `set_freq` 写入模式**，大大增强了模块的通用性。

### 📈 优化 (Improvements)

-   **[性能]** 优化了进程PID的获取方式，使用 `pidof` 命令替代原有逻辑，提升了 `AffinitySetter` (核心绑定) 等功能的执行效率和准确性。
-   **[调度]** 新增了针对 CFS (Completely Fair Scheduler) 调度器的优化参数，改善了任务在核心间的迁移和唤醒逻辑。
-   **[兼容性]** 使用 `busybox` 的 `wget` 代替 `curl` 来下载云控配置文件，提升了在部分缺少 `curl` 组件的设备上的兼容性。

## 🔗 相关链接

  - **GitHub 仓库**: [https://github.com/imacte/YukiCpuScheduler\_update](https://github.com/imacte/YukiCpuScheduler_update)
  - **Gitee 仓库**: [https://gitee.com/imacte\_ui/YukiCpuScheduler\_update](https://gitee.com/imacte_ui/YukiCpuScheduler_update)
  - **问题反馈**: [GitHub Issues](https://github.com/imacte/YukiCpuScheduler_update/issues)
