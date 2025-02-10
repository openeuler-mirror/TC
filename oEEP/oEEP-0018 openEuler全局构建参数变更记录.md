---
标题:     openEuler全局构建参数变更记录
类别:     信息整理
摘要:     全局构建参数变更记录
作者:     Funda Wang (fundawang at yeah.net)
状态:     活跃
编号:     oEEP-0018
创建日期: 2024-09-16
修订日期: 2025-02-08
---

## 背景说明

`openEuler-rpm-config` 包中含有被 openEuler 制品仓中所有软件的构建过程引用的全局配置。对 `openEuler-rpm-config` 的修改，会直接影响所有 openEuler 软件组件的维护。本文档旨在描述 `openEuler-rpm-config` 从 openEuler 24.09 版本开始的重要变化，这些变化对 openEuler 开发者可能的影响，以及开发者如何应对这些变化。

## 生效版本 24.09

### `LDFLAGS` 默认包含`-Wl,--as-needed`开关

**作用**：去除二进制文件中对非必需库的链接，加快程序冷启动时间。

**禁用说明**：不建议禁用此开关。

**参考链接**：  

- https://fedoraproject.org/wiki/Changes/RemoveExcessiveLinking
- https://wiki.gentoo.org/wiki/Project:Quality_Assurance/As-needed
- https://wiki.debian.org/ToolChain/DSOLinking#Only_link_with_needed_libraries

### 部分软件包的`CFLAGS`启用 Link Time Optimization (链接时优化)开关

**作用**：

自 24.09 起，[白名单](https://gitee.com/src-openeuler/openEuler-rpm-config/blob/openEuler-24.09/0001-Enable-LTO-By-Default.patch)中的软件包启用链接时优化(LTO)开关。这一开关将显著缩小二进制文件的大小，提高运行速度。

白名单外的软件包，`CFLAGS`不会添加 LTO 开关。maintainer 需要密切关注上游及其它发行版中 LTO 开关对本软件的影响，尽早评估其在 25.03 及后续版本中的适配情况。

**禁用说明**：

[白名单](https://gitee.com/src-openeuler/openEuler-rpm-config/blob/openEuler-24.09/0001-Enable-LTO-By-Default.patch)中的软件包，如确认开启 LTO 后软件产生副作用，应该首先积极联系上游，确认其解决意向。确认上游没有解决意向时，可联系 TC 或 Compiler SIG 寻求帮助以禁用此开关。

**参考链接**：

- https://gitee.com/src-openeuler/openEuler-rpm-config/blob/openEuler-24.09/0001-Enable-LTO-By-Default.patch
- https://fedoraproject.org/wiki/LTOByDefault
- https://wiki.gentoo.org/wiki/LTO
- https://wiki.debian.org/ToolChain/LTO

## 生效版本 25.03

### `CFLAGS`默认启用 Link Time Optimization (链接时优化)

**作用**：

根据TC会议决议，自 25.03 起全局启用链接时优化(LTO)开关，不再通过白名单控制开启 LTO 的软件包。这一开关将显著缩小二进制文件的大小，提高运行速度。

**禁用说明**：

仅当 LTO 开启后软件产生极大的副作用，且上游研发没有解决意向时，才可考虑禁用此开关。禁用时请在 spec 文件中定义 `%define _lto_cflags %{nil}`。

**参考链接**：

- https://fedoraproject.org/wiki/LTOByDefault
- https://wiki.gentoo.org/wiki/LTO
- https://wiki.debian.org/ToolChain/LTO

### 自动删除 libtool .la 文件

**作用**：

自动删除`libtool` `*.la`文件，这些文件只被`libtool`自己使用，其出现远早于 ELF 格式文件，目前已无任何作用。

**禁用说明**：

仅当程序运行中依赖`.la`文件(如调用了`lt_dlopen`函数)，或`.la`文件在下游软件编译时被深度引用时，才可考虑禁用此开关。禁用时请在 spec 文件中定义 `%define __brp_remove_la_files %{nil}`。

**参考链接**：

- https://fedoraproject.org/wiki/Changes/RemoveLaFiles
- https://projects.gentoo.org/qa/policy-guide/installed-files.html#pg0303
- https://wiki.debian.org/ReleaseGoals/LAFileRemoval

### 自动删除 /usr/share/info/dir 文件

**作用**：

`/usr/share/info/dir` 文件应该在RPM安装到最终用户计算机后，由计算机的本地程序生成，其中包含当前所有已安装 info 文件的目录。打包过程中生成此文件是错误的。

**禁用说明**：不需要禁用此开关。

### 打印 ELF 文件的符号引用分析说明

**作用**：

在打包结束后，打印对所有二进制 ELF 文件的符号引用分析说明。说明分为两种情况：

- 引用了多余的二进制库，其说明形式为“`unused libraries`”。
  - 此时开发者可研究分析链接语句的开关调用顺序，确保 `-Wl,--as-needed` 开关位于 `-lfoo` 二进制库之前。
- 引用了其它二进制库所提供的符号却没有链接到二进制库，其说明形式为“`undefined symbols`”
  - 此时开发者可研究提供相应符号的二进制库，将其加入链接语句。如确认所列的 .so 文件为主程序调用的插件模块，而有关符号由主程序提供，则相应报警可予以忽略。

**禁用说明**：

不需要禁用此开关，此特性只打印 Warning，不对打包过程的成功与否进行干预。

**参考链接**：

- https://wiki.mageia.org/en/Underlinking_issues_in_packaging
- https://wiki.mageia.org/en/Overlinking_issues_in_packaging
- https://wiki.debian.org/ToolChain/DSOLinking#Unresolved_symbols_in_shared_libraries

### Fortify Level 防御级别可设置

**作用**：当前默认构建选项中，包含级别为2的防御级别(Fortify level)，即在`CFLAGS`中生成`-Wp,-D_FORTIFY_SOURCE=2`。该选项旨在通过对 glibc 中的某些函数提供防御，以期拦截缓冲区溢出。考虑未来即将演进到级别为3的防御，以及个别软件包对此防御开关有进行屏蔽的需求，防御级别调整为软件包根据需要自行设置。

**禁用说明**：
系统的全局防御级别当前为2，与 openEuler 24.03 LTS 相比无变化。如软件包需要自定义防御级别或禁用防御，可自行在 spec 中增加下列语句：
```
%define _fortify_level 1 # 可降级为1级防御，即在 CFLAGS 中生成 -Wp,-U_FORTIFY_SOURCE,-D_FORTIFY_SOURCE=2
%define _fortify_level 0 # 可禁用防御，即不在 CFLAGS 生成 -D_FORTIFY_SOURCE 的开关
```
