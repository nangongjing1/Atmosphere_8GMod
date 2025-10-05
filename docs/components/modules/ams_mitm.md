# ams_mitm
此模块提供拦截其他系统模块提供的服务的方法。它根据目标服务进一步细分。

## bpc_mitm
bpc_mitm支持拦截电源控制服务的请求。当前拦截：
+ `am`系统模块（拦截覆盖菜单中的重启/电源按钮）
+ `fatal`系统模块（显著简化payload重启逻辑）
+ [nx-hbloader](https://github.com/switchbrew/nx-hbloader)（允许自制软件利用此功能）

## fs_mitm
fs_mitm支持拦截文件系统操作。它可以拒绝、延迟、替换或重定向对文件系统的任何请求。它使LayeredFS能够工作，从而允许替换游戏资源。

## hid_mitm
hid_mitm支持拦截控制器设备服务的请求。默认情况下当前已禁用。如果启用，它会拦截：
+ [nx-hbloader](https://github.com/switchbrew/nx-hbloader)（帮助自制软件无需因过去引入的重大变更而重新编译）

请注意，hid_mitm目前已弃用，未来可能会完全移除。

## ns_mitm
ns_mitm支持拦截应用程序控制服务的请求。当前拦截：
+ Web小应用程序（促进nx-hbloader网页浏览器启动）

## set_mitm
set_mitm支持拦截系统设置服务的请求。当前拦截：
+ `ns`系统模块和游戏（允许覆盖游戏区域设置）
+ 所有固件调试设置请求（允许修改未直接向用户公开的系统设置）

### 固件版本
set_mitm拦截`GetFirmwareVersion`命令，如果请求者是`qlaunch`或`maintenance`。
它修改返回的系统版本的`display_version`字段，导致版本在设置中显示为`#.#.#|AMS #.#.#|?`，其中`? = S`表示在系统eMMC下运行，`? = E`表示在模拟eMMC下运行。这使用户能够轻松验证他们运行的Atmosphère版本和eMMC环境。

### 系统设置
set_mitm为所有请求者拦截`GetSettingsItemValueSize`和`GetSettingsItemValue`命令。
这样做是为了启用系统设置的用户配置，这些设置在启动时从`/atmosphere/system_settings.ini`解析。有关系统设置格式的更多信息，请参见[此处](../../features/configurations.md)。

## dns_mitm
dns_mitm支持拦截DNS解析服务的请求，以启用重定向指定主机名的请求。

有关文档，请参见[此处](../../features/dns_mitm.md)。
