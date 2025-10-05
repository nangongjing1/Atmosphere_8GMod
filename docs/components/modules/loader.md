# loader
该模块是 Horizon OS 中 `ldr` 系统模块的重新实现，负责从可执行的 NSO 映像创建进程并注册其访问控制。

## 扩展功能
Atmosphère 扩展了此模块，允许通过存储在 SD 卡上的文件替换或修补可执行文件。请注意，某些服务需要 SD 卡访问，因此无法通过此方式替换或修补。

### Exefs 替换
Atmosphère 的重新实现允许替换文件系统中的可执行文件。

#### 分区替换
可以一次性用 PFS0 文件替换整个 exefs 分区。在这种情况下，Atmosphère 将加载以下文件：
```
/atmosphere/contents/<program id>/exefs.nsp
```

#### 文件替换
当创建进程时，loader 将在程序的 exefs 目录中搜索多个 NSO 文件名。
这些文件名按以下顺序：
  - rtld
  - main
  - subsdk0
  - subsdk1
  - ...
  - subsdk9
  - sdk

找到的每个 NSO 将被连续加载到进程中。进程的入口点位于第一个加载的 NSO，通常是 `rtld` 或 `main`。

此外，当加载进程时，loader 会在 exefs 目录中搜索 `main.npdm` 文件，该文件指定程序的权限。

Atmosphère 通过同时在 SD 卡上搜索这些文件来扩展此功能。当搜索文件时，loader 会首先检查它是否存在于 SD 卡上。如果存在，则使用该文件。否则，如果 exefs 中存在副本，则使用该副本。将搜索以下目录：
```
/atmosphere/contents/<program id>/exefs/
```

这允许用自制版本替换 applet、系统模块甚至游戏。

##### 文件存根
为了防止即使 exefs 中存在 NSO 文件也被加载，loader 还会检查是否存在存根文件。如果存在此类文件，则不会加载 NSO。文件应命名为 `rtld.stub`、`main.stub` 等，并且可以为空。

##### 技术语义
loader 的内容覆盖语义（如您从上述内容可能观察到的）可能难以理解。以下是 loader 在尝试读取程序 ID 的文件时决定读取内容的非常技术性的语义的简要描述。

* 如果程序 ID 存在外部内容文件系统，则直接使用外部内容文件系统，无需进一步重定向。
* 否则，如果程序 ID 被 [nx-hbloader](https://github.com/switchbrew/nx-hbloader/releases) 覆盖（见下文自制软件支持），则直接使用 hbl 的 nsp 文件系统，无需进一步重定向。
* 否则，如果为程序 ID 启用了内容重定向（由可配置的按钮组合控制）且 SD 卡上存在松散文件，则使用该松散文件。
* 否则，如果存在存根文件，则返回"未找到"错误。
* 否则，如果存在 SD 卡可执行文件系统（"exefs.nsp"），则直接使用它。
* 最后，直接使用"真实"/基础代码文件系统，无需进一步重定向。

此外，还有一些与 Atmosphere 重定向相关的其他技术细节：
* 当使用 nx-hbloader 覆盖时，真实代码文件系统必须存在。当读取"main.npdm"（程序能力描述符文件）时，会读取真实代码文件系统中的内容，以确定覆盖的是 applet 还是应用程序。这使 nx-hbloader 能自动支持 applet 和应用程序环境。
* 当覆盖应用程序时，真实代码文件系统必须存在并包含有效内容。这是执行符合任天堂标准的内容验证过程所必需的。
* 当启动程序时，启动请求者会同时指定程序 ID 和"存储 ID"。当指定的存储 ID 为"none"（通常始终无效）时，Atmosphere 假定正在尝试启动自定义系统模块。这移除了上述对基础内容有效性的要求；但仍使用上述过程确定如何重定向内容，但如果真实/基础代码文件系统不存在，则对"真实"/基础代码文件系统的读取可能返回"未找到"错误。

### NSO 修补
当加载 NSO 时，Atmosphère 的重新实现将在 SD 卡的以下位置搜索 IPS 补丁文件：
```
/atmosphere/exefs_patches/<patchset name>/<nso build id>.ips
```

这种组织方式允许影响多个 NSO 的补丁集作为单个目录分发，并允许多个补丁集的补丁堆叠应用。将在每个补丁集目录中搜索补丁文件。每个补丁文件的名称应与要影响的 NSO 的十六进制构建 ID 匹配，但可以省略尾随的零字节。由于 NSO 构建 ID 对每个 NSO 都是唯一的，这意味着补丁只会应用于它们预期的文件。

补丁文件接受 IPS 格式或 IPS32 格式。

由于 NSO 文件是压缩的，补丁文件不是在压缩 NSO 的原始版本和修改版本之间制作的，而是在未压缩版本的 NSO 和修改后（仍为未压缩）版本的 NSO 之间制作的。这也意味着补丁文件无法手动应用于压缩版本的 NSO；必须应用于未压缩版本。Atmosphère 的重新实现在加载进程时将正确应用这些补丁，无论它找到的 NSO 是否压缩。

在制作补丁时，可以使用 [hactool](https://github.com/SciresM/hactool) 查找 NSO 的构建 ID 并解压缩 NSO。最新版本的 [ReSwitched IDA 加载器](https://github.com/reswitched/loaders) 可用于将未压缩的 NSO 加载到 IDA 中，以便您可以[对输入文件应用补丁](https://www.hex-rays.com/products/ida/support/idadoc/1618.shtml)。然后，可以使用任何 IPS 工具在原始 NSO 和修补后的 NSO 之间创建补丁。请注意，如果您要修补的 NSO 大于 16 MiB，则必须使用支持 IPS32 的工具。

### 自制软件支持
Atmosphère 为 [nx-hbloader](https://github.com/switchbrew/nx-hbloader/releases) 和 [nx-hbmenu](https://github.com/switchbrew/nx-hbmenu/releases) 提供一流支持。

nx-hbloader 进程的启动由可配置的按钮输入控制。更多详细信息请参见[此处](../../features/configurations.md)。

此外，loader 还进行了扩展，使自制软件能够启动 Web applet。这通常要求启动 applet 的应用程序在已安装的 NCA 中包含 HTML 手册内容。Atmosphère 的重新实现将自动确保用于检查此功能的命令成功，并将相关文件系统重定向到 `/atmosphere/hbl_html/` 子目录。

### IPC 命令
Atmosphère 的重新实现通过几个自定义命令扩展了 HIPC loader 服务的 API。

`ldr:pm` 扩展命令的 SwIPC 定义如下：
```
interface ams::ldr::pm::ProcessManagerInterface is ldr:pm {
  ...
  [65000] AtmosphereHasLaunchedProgram(ncm::ProgramId program_id) -> sf::Out<bool> out;
  [65001] AtmosphereGetProgramInfo(ncm::ProgramLocation &loc) -> sf::Out<ProgramInfo> out_program_info, sf::Out<cfg::OverrideStatus> out_status;
  [65002] AtmospherePinProgram(ncm::ProgramLocation &loc, cfg::OverrideStatus &override_status) -> sf::Out<PinId> out_id;
}
```

`ldr:dmnt` 扩展命令的 SwIPC 定义如下：
```
interface ams::ldr::dmnt::DebugMonitorInterface is ldr:dmnt {
  ...
  [65000] AtmosphereHasLaunchedProgram(ncm::ProgramId program_id) -> sf::Out<bool> out;
}
```

`ldr:shel` 扩展命令的 SwIPC 定义如下：
```
interface ams::ldr::shell::ShellInterface is ldr:shel {
  ...
  [65000] AtmosphereRegisterExternalCode(ncm::ProgramId program_id) -> sf::OutMoveHandle out;
  [65001] AtmosphereUnregisterExternalCode(ncm::ProgramId program_id);
}
```
