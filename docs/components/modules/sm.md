# sm
该模块是对 Horizon OS 的 `sm` 系统模块的重新实现，负责服务管理。

## 扩展功能
Atmosphère 通过额外的 IPC 命令和新服务扩展了此模块。

### 调试监视器
Atmosphère 的重新实现提供了一个接口 `sm:dmnt`，允许调试监视器查询服务管理器的状态。

`sm:dmnt` 的 SwIPC 定义如下：
```
interface ams::sm::DmntService is sm:dmnt {
  [65000] AtmosphereGetRecord(ServiceName service) -> sf::Out<ServiceRecord> record;
  [65001] AtmosphereListRecords(u64 offset) -> sf::OutArray<ServiceRecord> &records, sf::Out<u64> out_count;
  [65002] AtmosphereGetRecordSize() -> sf::Out<u64> record_size;
}
```

### IPC 命令
Atmosphère 的重新实现通过几个自定义命令扩展了 HIPC 加载器服务的 API。

`sm:` 扩展命令的 SwIPC 定义如下：
```
interface ams::sm::UserService is sm: {
  ...
  [65000] AtmosphereInstallMitm(ServiceName service) -> sf::OutMoveHandle srv_h, sf::OutMoveHandle qry_h;
  [65001] AtmosphereUninstallMitm(ServiceName service);
  [65002] Deprecated_AtmosphereAssociatePidTidForMitm();
  [65003] AtmosphereAcknowledgeMitmSession(ServiceName service) -> sf::Out<MitmProcessInfo> client_info, sf::OutMoveHandle fwd_h;
  [65004] AtmosphereHasMitm(ServiceName service) -> sf::Out<bool> out;
  [65005] AtmosphereWaitMitm(ServiceName service);
  [65006] AtmosphereDeclareFutureMitm(ServiceName service);
  
  [65100] AtmosphereHasService(ServiceName service) -> sf::Out<bool> out;
  [65101] AtmosphereWaitService(ServiceName service);
}
```

`sm:m` 扩展命令的 SwIPC 定义如下：
```
interface ams::sm::ManagerService is sm:m {
  ...
  [65000] AtmosphereEndInitDefers(os::ProcessId process_id, sf::InBuffer &acid_sac, sf::InBuffer &aci_sac);
  [65001] AtmosphereHasMitm(ServiceName service) -> sf::Out<bool> out;
  [65002] AtmosphereRegisterProcess(os::ProcessId process_id, ncm::ProgramId program_id, cfg::OverrideStatus override_status, sf::InBuffer &acid_sac, sf::InBuffer &aci_sac);
}
