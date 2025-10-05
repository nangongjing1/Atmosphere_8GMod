# 构建 Atmosphère
构建 Atmosphère 是一个非常直接的过程，几乎完全依赖于 [devkitPro](https://devkitpro.org) 组织提供的工具。

## 依赖项
+ [devkitA64](https://devkitpro.org)
+ [devkitARM](https://devkitpro.org)
+ [Python 2](https://www.python.org) (Python 3 可能也可以工作，但不保证)
+ [LZ4](https://pypi.org/project/lz4)
+ [PyCryptodome](https://pypi.org/project/pycryptodome) (可选)
+ [hactool](https://github.com/SciresM/hactool)

## 说明
1. 按照[此指南](https://devkitpro.org/wiki/Getting_Started)安装和配置构建过程所需的所有工具。

2. 通过 (dkp-)pacman 安装以下软件包：
    + `switch-dev`
    + `switch-glm`
    + `switch-libjpeg-turbo`
    + `devkitARM`
    + `devkitarm-rules`
    + `hactool`

3. 通过 Python 的包管理器 `pip` 安装以下库（由 [exosphere](components/exosphere.md) 所需）：
    + `lz4`

4. 最后，克隆 Atmosphère 仓库并在其根目录下运行 `make`。
