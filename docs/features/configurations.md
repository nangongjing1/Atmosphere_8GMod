# 配置
Atmosphère 提供多种可自定义的配置，以更好地适应用户需求。

## stratosphere.ini
这是 fusée 用于配置用户空间系统模块的配置文件。
该文件位于 SD 卡的 `/atmosphere/config/` 文件夹下，默认模板可在 `/atmosphere/config_templates/` 文件夹中找到。

### 配置 "nogc" 保护
"nogc" 是由 fusée-secondary 提供的功能，用于禁用 Nintendo Switch 的游戏卡读取器。其目的是在主机从较低固件版本（特别是 4.0.0 或 9.0.0 固件版本，这些版本引入了游戏卡读取器固件的更新）升级时，防止读取器被更新。默认情况下，Atmosphère 会自动保护游戏卡读取器，但您可以自由更改此设置。

要更改其功能，请在 `stratosphere` 部分添加以下行，并根据以下列表更改 `X` 的值：
```
[stratosphere]
nogc = X
```
```
1 = 强制启用 nogc，因此 Atmosphère 将始终禁用游戏卡读取器。
0 = 强制禁用 nogc，因此 Atmosphère 将始终启用游戏卡读取器。
```

## 添加自定义启动画面
Atmosphère 提供自己的默认启动画面，在启动时显示。但是，这可以随意替换。

启动画面分辨率必须为 1280x720。

源代码树中提供了一个脚本 (`/utilities/insert_splash_screen.py`)，用于将自定义启动画面插入到发布版二进制文件中。

执行以下命令使用该脚本：
`python insert_splash_screen.py <自定义启动画面图像路径> <SD卡上/atmosphere/package3的路径>`

## emummc.ini
这是用于 [emummc](../components/emummc.md) 组件的配置文件。
该文件位于 SD 卡的 `/emuMMC/` 文件夹下。

请参考项目仓库 [此处](https://github.com/m4xw/emuMMC) 获取详细说明和文档。

## exosphere.ini
这是 exosphère 使用的配置文件。
该文件位于 SD 卡根目录，默认模板可在 `/atmosphere/config_templates/` 文件夹中找到。

### 配置调试模式
默认情况下，Atmosphère 向 Horizon 内核发出调试已启用的信号，但禁用用户模式调试，这可能导致不良副作用。如果您希望更改此行为，请转到 `exosphere` 部分，并根据以下列表更改 `X` 的值。
```
[exosphere]
debugmode = X
debugmode_user = X
```
```
1 = 启用
0 = 禁用
```

### 清空 PRODINFO
Atmosphère 提供了一种方式，允许用户在模拟或系统 eMMC 环境中清空其出厂安装的校准数据（称为 PRODINFO）。您可以在相应的模板文件中找到有关此功能的更详细信息。不建议使用此配置。

## override_config.ini
该文件位于 SD 卡的 `/atmosphere/config/` 文件夹下，默认模板可在 `/atmosphere/config_templates/` 文件夹中找到。

### 覆盖格式
覆盖配置在启动过程中从 `/atmosphere/config/override_config.ini` 文件解析。

默认情况下 `override_config.ini` 未配置。它可用于选择某些按钮的行为，并将其绑定到功能，例如启动自制菜单或启用金手指代码管理器。

您可以使用以下有效按钮列表修改 `override_config.ini` 中的 override_key 条目：
| Formal Name | .ini Name |
| ----------- | --------- |
| A Button    | A         |
| B Button    | B         |
| X Button    | X         |
| Y Button    | Y         |
| Left Stick  | LS        |
| Right Stick | RS        |
| L Button    | L         |
| R Button    | R         |
| ZL Button   | ZL        |
| ZR Button   | ZR        |
| + Button    | PLUS      |
| - Button    | MINUS     |
| Left Dpad   | DLEFT     |
| Up Dpad     | DUP       |
| Right Dpad  | DRIGHT    |
| Down Dpad   | DDOWN     |
| SL Button   | SL        |
| SR Button   | SR        |

要反转覆盖键的行为，请在您希望使用的按钮前放置感叹号。这样将在按住该按钮时启动实际游戏，而不是进入自制菜单。例如，`override_key=!R` 将在启动时按住 R 键运行游戏，否则将启动到自制菜单。之后，您可以将 SD 卡重新插入 Switch 并像往常一样启动到 Atmosphère。现在，您应该能够通过启动您指定的程序选择进入自制菜单。

## system_settings.ini
该文件位于 SD 卡的 `/atmosphere/config/` 文件夹下，默认模板可在 `/atmosphere/config_templates/` 文件夹中找到。

### 设置格式
Atmosphère 提供了一种覆盖系统使用的固件调试设置的方法。这些设置可以在启动过程中从 `/atmosphere/config/system_settings.ini` 文件解析。该文件是一个普通的 ini 文件，但有一些特定的解释规则。

设置标识符的标准表示形式为 `name!key`。在 `system_settings.ini` 中，这表示为 `name` 部分，其中包含 `key` 条目。例如：
```
[name]
key = ...
```

设置可以具有变量类型（字符串、整数值、字节数组等）。为适应这一点，`system_settings.ini` 必须将值存储为 `type_identifier!value_store` 对。支持多种不同的类型，其标识符如下所述。
请注意，格式错误的值字符串将导致启动时发生致命错误。下面给出了自定义设置的完整示例（设置 `eupld!upload_enabled = 0`）：
```
[eupld]
upload_enabled = u8!0x0
```

#### 支持的类型
* 字符串
    * 类型标识符: `str`, `string`
    * 值字符串直接用作设置，并附加空终止符。
* 整型
    * 类型标识符: `u8`, `u16`, `u32`, `u64`
    * 值字符串通过调用 `strtoul(value, NULL, 0)` 解析。
    * 设置位宽由标识符确定（1字节为8位，2字节为16位，依此类推）。
* 原始字节
    * 类型标识符: `hex`, `bytes`
    * 值字符串被解析为十六进制字符串。
        * 值字符串长度必须为偶数，否则解析时将抛出致命错误。

## 内容特定标志
Atmosphère 支持根据 SD 卡上存在的 `flags` 自定义 CFW 行为。

以下标志支持基于每个程序进行配置，方法是将 `<flag_name>.flag` 文件放入 `/atmosphere/contents/<program_id>/flags/` 目录：
+ `boot2`，表示该程序应在 `boot2` 过程中启动。
+ `redirect_save`，表示该程序希望将其保存数据重定向到 SD 卡。
