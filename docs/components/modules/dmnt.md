# dmnt
该模块是 Horizon OS 中 `dmnt` 系统模块的重新实现，负责提供调试监视器（debug monitor）功能。

## 扩展功能
Atmosphère 实现了一个扩展，用于提供金手指（cheat code）功能。

### 金手指服务（Cheat Service）
通过服务 `dmnt:cht` 提供了一个 HIPC 服务 API，用于与金手指管理器交互。有关金手指格式的更多信息，请参见[这里](../../features/cheats.md)。

`dmnt:cht` 的 SwIPC 定义如下：
```
interface ams::dmnt::cheat::CheatService is dmnt:cht {
  [65000] HasCheatProcess() -> sf::Out<bool> out;
  [65001] GetCheatProcessEvent() -> sf::OutCopyHandle out_event;
  [65002] GetCheatProcessMetadata() -> sf::Out<CheatProcessMetadata> out_metadata;
  [65003] ForceOpenCheatProcess();
  [65004] PauseCheatProcess();
  [65005] ResumeCheatProcess();

  [65100] GetCheatProcessMappingCount() -> sf::Out<u64> out_count;
  [65101] GetCheatProcessMappings(u64 offset) -> sf::OutArray<MemoryInfo> &mappings, sf::Out<u64> out_count;
  [65102] ReadCheatProcessMemory(u64 address, u64 out_size) -> sf::OutBuffer &buffer;
  [65103] WriteCheatProcessMemory(sf::InBuffer &buffer, u64 address, u64 in_size);
  [65104] QueryCheatProcessMemory(u64 address) -> sf::Out<MemoryInfo> mapping;

  [65200] GetCheatCount() -> sf::Out<u64> out_count;
  [65201] GetCheats(u64 offset) -> sf::OutArray<CheatEntry> &cheats, sf::Out<u64> out_count;
  [65202] GetCheatById(u32 cheat_id) -> sf::Out<CheatEntry> cheat;
  [65203] ToggleCheat(u32 cheat_id);
  [65204] AddCheat(CheatDefinition &cheat, bool enabled) -> sf::Out<u32> out_cheat_id;
  [65205] RemoveCheat(u32 cheat_id);
  [65206] ReadStaticRegister(u8 which) -> sf::Out<u64> out;
  [65207] WriteStaticRegister(u8 which, u64 value);
  [65208] ResetStaticRegisters();

  [65300] GetFrozenAddressCount() -> sf::Out<u64> out_count;
  [65301] GetFrozenAddresses(u64 offset) ->sf::OutArray<FrozenAddressEntry> &addresses, sf::Out<u64> out_count;
  [65302] GetFrozenAddress(u64 address) -> sf::Out<FrozenAddressEntry> entry;
  [65303] EnableFrozenAddress(u64 address, u64 width) -> sf::Out<u64> out_value;
  [65304] DisableFrozenAddress(u64 address);
}
```
