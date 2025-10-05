# DNS.mitm
自 0.18.0 版本起，atmosphère 提供了一种重定向 DNS 解析请求的机制。

默认情况下，atmosphère 会将官方遥测服务器的解析请求重定向到环回地址。

## Hosts 文件

DNS.mitm 可通过使用略微扩展的 `hosts` 文件格式进行配置，该文件仅在系统启动时解析一次。

具体来说，DNS.mitm 解析的 hosts 文件具有以下扩展特性：
+ `*` 被视为通配符，可匹配主机名中任意位置的 0 个或多个字符
+ `%` 作为 `nsd!environment_identifier` 值的占位符。在生产设备上，该值始终为 `lp1`

如果 hosts 文件中有多个条目匹配某个域名，则使用最后定义的匹配项。

请注意，自制程序可通过向连接的 `sfdnsres` 会话发送扩展 IPC 命令 65000 ("AtmosphereReloadHostsFile") 来触发 hosts 文件重新解析。

### Hosts 文件选择

Atmosphère 将按以下顺序尝试读取 hosts 文件，并在成功执行文件读取后停止：

+ (仅 emummc) `/atmosphere/hosts/emummc_%04lx.txt`，格式化为 emummc 的 ID 号（参见 `emummc.ini`）
+ (仅 emummc) `/atmosphere/hosts/emummc.txt`
+ (仅 sysmmc) `/atmosphere/hosts/sysmmc.txt`
+ `/atmosphere/hosts/default.txt`

如果 `/atmosphere/hosts/default.txt` 不存在，atmosphère 将创建该文件并包含默认内容。

### Atmosphère 默认设置

默认情况下，atmosphère 的默认重定向规则会**附加到**已加载 hosts 文件的内容之后。

这相当于将 atmosphère 的默认设置前置到加载的 hosts 文件中。

此设置被认为是可取的，因为如果用户在系统更新后忘记更新自定义 hosts 文件（而更新更改了遥测服务器），它可以最大限度地减少遥测风险。

可以通过在 `system_settings.ini` 中设置 `atmosphere!add_defaults_to_dns_hosts = u8!0x0` 来退出此行为。

当前的默认重定向规则为：

```
# Nintendo telemetry servers
127.0.0.1 receive-%.dg.srv.nintendo.net receive-%.er.srv.nintendo.net
```

## 调试

在启动时（或重新解析 hosts 文件时），DNS.mitm 会将其选择的 hosts 文件以及解析的所有重定向内容记录到 `/atmosphere/logs/dns_mitm_startup.log`。

此外，如果用户在 `system_settings.ini` 中设置 `atmosphere!enable_dns_mitm_debug_log = u8!0x1`，DNS.mitm 会将所有 GetHostByName/GetAddrInfo 请求记录到 `/atmosphere/logs/dns_mitm_debug.log`。所有重定向发生时都会被记录。

## 完全退出 DNS.mitm
如需完全禁用 DNS.mitm，可编辑 `system_settings.ini` 并设置 `atmosphere!enable_dns_mitm = u8!0x0`。
