# fatal
该模块是 Horizon OS 中 `fatal` 系统模块的重新实现，负责管理致命错误报告。

Atmosphère 的重新实现会阻止错误报告创建，并绘制一个自定义错误屏幕，显示寄存器和回溯信息。它还会尝试收集任何和所有崩溃的调试信息，并尝试将报告保存到 SD 卡的 `/atmosphere/fatal_reports/` 文件夹下。
