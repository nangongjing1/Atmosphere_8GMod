# creport
该模块是 Horizon OS 中 `creport` 系统模块的重新实现，负责管理崩溃报告。

Atmosphère 的重新实现将生成的崩溃报告重定向写入到 SD 卡的 `/atmosphere/crash_reports/` 文件夹下，并阻止自动上传这些崩溃报告。
