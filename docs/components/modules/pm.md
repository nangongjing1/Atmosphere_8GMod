# pm
该模块是对 Horizon OS 的 `pm` 系统模块的重新实现，负责跟踪系统上运行的进程，并管理资源限制。

## 扩展功能
Atmosphère 通过额外的 IPC 命令和内存限制更改扩展了此模块。

### IPC 命令
Atmosphère 的重新实现通过几个自定义命令扩展了 HIPC 加载器服务的 API。

`pm:dmnt` 扩展命令的 SwIPC 定义如下：
```
interface ams::pm::dmnt::DebugMonitorServiceBase is pm:dmnt {
  ...
  [65000] AtmosphereGetProcessInfo(os::ProcessId process_id) -> sf::OutCopyHandle out_process_handle, sf::Out<ncm::ProgramLocation> out_loc, sf::Out<cfg::OverrideStatus> out_status;
  [65001] AtmosphereGetCurrentLimitInfo(u32 group, u32 resource) -> sf::Out<s64> out_cur_val, sf::Out<s64> out_lim_val;
}
```

`pm:info` 扩展命令的 SwIPC 定义如下：
```
interface ams::pm::info::InformationService is pm:info {
  ...
  [65000] AtmosphereGetProcessId(ncm::ProgramId program_id) -> sf::Out<os::ProcessId> out;
  [65001] AtmosphereHasLaunchedProgram(ncm::ProgramId program_id) -> sf::Out<bool> out;
  [65002] AtmosphereGetProcessInfo(os::ProcessId process_id) -> sf::Out<ncm::ProgramLocation> out_loc, sf::Out<cfg::OverrideStatus> out_status;
}
```

### 额外的系统内存
Atmosphère 的重新实现默认将 APPLET 内存池缩小 24 MiB，并将这些内存分配给 SYSTEM 池。这使得自定义系统模块可以使用更多内存，而不会达到 SYSTEM 内存限制。
