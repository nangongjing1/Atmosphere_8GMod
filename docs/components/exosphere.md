# exosphère
exosphère是Horizon OS安全监视器（Secure Monitor）的定制化重新实现。
安全监视器遵循Arm的TrustZone相同的设计原则，在此上下文中这两个术语可以互换使用。它在主处理器可用的最高权限模式（EL3）下运行，负责系统所需的所有敏感加密操作以及每个CPU的电源管理。

## 扩展
exosphère通过提供自制软件生态系统所需的自定义SMC（安全监视器调用）扩展了原始安全监视器设计。目前包括：
```
uint32_t smc_ams_iram_copy(smc_args_t *args);
uint32_t smc_ams_write_address(smc_args_t *args);
uint32_t smc_ams_get_emummc_config(smc_args_t *args);
```

此外，exosphère扩展了Horizon OS提供的两个用于获取/设置配置项的SMC功能。exosphère提供以下自定义配置项：
```
CONFIGITEM_EXOSPHERE_VERSION = 65000,
CONFIGITEM_NEEDS_REBOOT = 65001,
CONFIGITEM_NEEDS_SHUTDOWN = 65002,
CONFIGITEM_EXOSPHERE_VERHASH = 65003,
CONFIGITEM_HAS_RCM_BUG_PATCH = 65004,
CONFIGITEM_SHOULD_BLANK_PRODINFO = 65005,
CONFIGITEM_ALLOW_CAL_WRITES = 65006,
```

### smc_ams_iram_copy
此函数实现在DRAM和IRAM之间复制最多一个页面的数据。其参数为：
```
args->X[1] = DRAM地址（由内核转换），必须4字节对齐。
args->X[2] = IRAM地址，必须4字节对齐。
args->X[3] = 大小（必须 <= 0x1000 且4字节对齐）。
args->X[4] = 0表示读取，1表示写入。
```

### smc_ams_write_address
此函数实现对DRAM页面的写入。其参数为：
```
args->X[1] = 虚拟地址，必须按大小字节对齐且EL0可读。
args->X[2] = 值。
args->X[3] = 大小（必须为1、2、4或8）。
```

### smc_ams_get_emummc_config
此函数检索当前[emummc](emummc.md)上下文的配置。其参数为：
```
args->X[1] = MMC ID，必须按大小字节对齐且EL0可读。
args->X[2] = 输出指针（用于基于文件的路径 + nintendo目录），必须至少0x100字节。
```

### CONFIGITEM_EXOSPHERE_VERSION
此自定义配置项获取当前exosphere版本信息。

### CONFIGITEM_NEEDS_REBOOT
此自定义配置项用于发起系统重启进入RCM或利用辅助漏洞实现热启动代码执行的warmboot payload。

### CONFIGITEM_NEEDS_SHUTDOWN
此自定义配置项用于发起系统关机，利用辅助漏洞实现热启动代码执行的warmboot payload。

### CONFIGITEM_EXOSPHERE_VERHASH
此自定义配置项获取当前exosphere的git提交哈希。

### CONFIGITEM_HAS_RCM_BUG_PATCH
此自定义配置项获取设备是否已修复CVE-2018-6242漏洞。

### CONFIGITEM_SHOULD_BLANK_PRODINFO
此自定义配置项获取设备是否应模拟"空白"的PRODINFO。更多信息请参见[此处](../features/configurations.md)。

### CONFIGITEM_ALLOW_CAL_WRITES
此自定义配置项获取设备是否应允许写入校准分区。

## lp0fw
这是一个小型内置payload，负责在热启动期间唤醒系统。

## sc7fw
这是一个小型内置payload，负责在热启动期间使系统进入睡眠状态。

## rebootstub
这是一个小型内置payload，提供将系统重启到任意选定payload的功能。
