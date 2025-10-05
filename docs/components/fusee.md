# fusée
fusée是用于启动Atmosphère环境的自定义引导加载程序。

## fusée
fusée是Atmosphère代码中第一个在硬件上运行的部分。
它作为独立的payload分发，旨在通过利用CVE-2018-6242漏洞在RCM模式下启动。

此payload负责任天堂Switch所需的所有底层硬件初始化，设置加密系统，挂载/模拟eMMC，注入/修补系统模块，并启动exosphere组件。
