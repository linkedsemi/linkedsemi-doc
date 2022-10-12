# Getting Started

## Prerequisites

### Hardware

- J-Link
- Evaluation Board
- PC 

### Software Development Kit

​	任选一个链接将SDK下载下来，推荐使用gitee，下载速度更快

- gitee链接：<https://gitee.com/linkedsemi/LS_SDK>
    - [v1.1.0 Release](https://gitee.com/linkedsemi/LS_SDK/releases/tag/v1.1.0)
    - [v1.0.0 Release](https://gitee.com/linkedsemi/LS_SDK/releases/tag/v1.0.0)
- github链接：<https://github.com/linkedsemi/LS_SDK>
    - [v1.1.0 Release](https://github.com/linkedsemi/LS_SDK/releases/tag/v1.1.0)
    - [v1.0.0 Release](https://github.com/linkedsemi/LS_SDK/releases/tag/v1.0.0)

### Development Environment

- Supported Compiler Toolchains
    - GCC + VSCode + Python
    - Keil(MDK)

- Supported Debugger Host
    - J-Link Software

You can choose any one from the supported toolchains for building your program.

Debugger host is used for communication with the debug probe.

#### J-Link Software
[J-Link Software Installation & Configuration](./getting_started/jlink)

#### Keil
[Keil Environment Setup](./getting_started/keil)

#### VSCode
[VSCode Environment Setup](./getting_started/vscode)


## Build/Program/Debug

### Keil
[Develop by Keil](./getting_started/keil)
### VSCode
[Develop by VSCode](./getting_started/vscode)


```{toctree}
:hidden:
getting_started/keil
getting_started/vscode
getting_started/jlink
```