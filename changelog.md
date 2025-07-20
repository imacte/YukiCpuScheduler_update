# 🚀 YukiCpuScheduler v2.1.0 更新日志

> 💡 **小贴士**: 模块会自动为你匹配最佳的性能配置文件。想获得极致体验？可以随时在 `/data/adb/modules/YukiCpuScheduler/configs/` 目录下手动调整 `config.yaml` 和 `thread.yaml`。

## 📋 更新详情

### 🐛 **问题修复 (Bug Fixes)**

1.  **修复了一个严重的核心功能Bug**
    *   修复了工具函数 `popenRead` 中因错误使用 `sizeof` 导致的逻辑缺陷。此Bug会导致该函数无法正确读取超过8个字节的进程PID列表。
    *   这个问题曾可能引起核心绑定功能（`AffinitySetter`）在处理多个或长PID的系统服务时失效。
    *   修复后的函数现在能够**安全、高效地处理任意长度的PID**，确保了核心绑定功能的可靠性。

## 🔗 相关链接

  - **GitHub 仓库**: [https://github.com/imacte/YukiCpuScheduler\_update](https://github.com/imacte/YukiCpuScheduler_update)
  - **Gitee 仓库**: [https://gitee.com/imacte\_ui/YukiCpuScheduler\_update](https://gitee.com/imacte_ui/YukiCpuScheduler_update)
  - **问题反馈**: [GitHub Issues](https://github.com/imacte/YukiCpuScheduler_update/issues)
