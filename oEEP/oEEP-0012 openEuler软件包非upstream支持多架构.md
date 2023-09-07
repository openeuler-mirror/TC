---
标题:     openEuler软件包非upstream支持多架构
类别:     流程设计
摘要:     制品仓软件包的非upstream支持
作者:     叶青龙 (yeqinglong@kylinsec.com.cn) 胡欣蔚 (shinwell_hu@openeuler.sh)
状态:     初始化
编号:     oEEP-0012
创建日期:  2023-08-17
修订日期:  2023-08-30
---

## 动机/问题描述:

### 初衷
openEuler 版本中支持众多体系架构，其中不乏一些软件组件，在 upstream 上还没有完整支持对应的架构；或者 upstream 已经支持，但是 openEuler 特定版本中选用的特定软件版本尚未完整支持对应的架构。
我们期望：
- 无论这些体系架构支持在 upstream 中是否已经具备，openEuler 有能力使能
- 不同体系架构在 openEuler 中应共享一份代码，即不同架构在 rpmbuild prepare 之后的代码是一致的，以保证所有的代码逻辑漏洞修复在所有架构上可落实
- 在体系架构使能代码未被 upstream 接纳的情况下，openEuler 的软件包选型演进不应该被阻拦
- 在体系架构使能代码已经 upstream 的情况下，openEuler 的软件包可以快速通过升级等手段跟随，不再独立维护体系架构使能代码
为了能够在 openEuler 制品仓中完整支持所有的体系架构，并且长期可维护，社区需要建立相应的规范和流程要求。

### 当前问题
社区当前架构相关支持的补丁缺少约束，不同的软件包管理时接纳架构相关补丁没有统一的要求，包括但不限于：
  - 不同架构在同一个地方反复修改，造成不必要的依赖复杂度，难以维护
  - 按架构打补丁，使得不同架构的构建用代码不一致，存在代码到二进制一致性风险
  - 补丁没有区分 upstream backport 和开发中，软件包演进时分析困难
  - 架构相关补丁和其他补丁没有明显区分，并且在不同软件包中应用的顺序各不相同

### 相关SIG组及职责
- 所有 SIG：SIG 的PR review checklist中应该增加与架构补丁合入相关的要求，并且能够在PR review过程中落实；
- openEuler Infra SIG：增强CI，提升对构建过程中的 ifarch 等条件构建宏的敏感性

## 方案的详细描述:
### 对代码的要求
1. 原则上不按架构分别打补丁，rpm spec 文件中出现 ifarch 宏的，必须在之前一行通过注释的方式明确说明原因和当前技术限制。
2. 补丁名称显式说明是否已经被 upstream 接纳，或来自某个特定版本的backport，便于版本升级时处理
```
Patch2:		backport-fiels-version-info.patch
```
3. 原则上新增架构相关代码文件，和修改架构相关代码的补丁应明确分开，不在同一个补丁中既修改代码，又新增文件，比如：
```
Patch2:		0002-add-riscv-support-not-upstream-modified-files.patch
Patch3:		0003-add-riscv-support-not-upstream-new-files.patch
```
4. 对于修改架构相关代码的补丁，一个补丁尽量不要包含太多文件修改，尽量拆分，可使用 split-patch 工具，一个补丁最多包括5个文件修改（暂定），如：
```
Patch1001:	1001-add-riscv-support-not-upstream-modified-files-1.patch
Patch1002:  1002-add-riscv-support-not-upstream-modified-files-2.patch
```
5. 对于同一个文件的不同架构补丁，原则上按架构相对稳定程度排序，优先应用架构稳定的补丁。不同软件包维护者可根据实际情况选择，但在不同文件的补丁应用中应保持一致。
### 补丁应用顺序
**注意** : rpm 中补丁PatchXXXX中的XXXX序号不会影响应用顺序，如下是错误的展示：
```rpm
Patch0002: patch-2.diff
Patch0001: patch-1.diff
```
实际应用时会先打 patch-2.diff，然后打 patch-1.diff。所以 openEuler 要求所有补丁声明的顺序必须与补丁序号保持一致。正确的声明顺序是：
```
Patch0001: patch-1.diff
Patch0002: patch-2.diff
```
### 补丁声明顺序
1. 所有来自同一版本 upstream 的补丁声明范围为 0001-0999
2. 所有架构相关的补丁声明范围为 1000 - 1999
3. 预留补丁编号范围 2000 - 2999
4. 所有CVE漏洞修复及上游其他版本backport 补丁，声明范围为 3000 - 4999
5. 所有 openEuler 特性补丁声明范围为 5000 -

### 软件包修复CVE
CVE 类补丁声明在架构补丁之后，天然要求 openEuler 漏洞修复要考虑对所有体系架构的支持。如果需要，CVE补丁应针对 openEuler 支持的所有架构做适配刷新。这在架构补丁与CVE同时修改同一个文件时可以通过补丁冲突直观检出。但存在一种可能，CVE修复了其他所有架构的独立文件，这时 maintainer 需要联系架构补丁提交者，确认修复方案。如果特定架构的修复难度过大，影响其他架构的补丁发布节奏，可以通过 ifarch 在构建时临时规避。此时 ifarch 可以作为 CI 问题，持续提醒 maintainer 和 committer 解决改问题。长期存在此类问题的架构不适宜作为 openEuler LTS 版本正式支持对象。

### 软件包升级
所有 upstream 补丁在架构声明之前，天然要求upstream开发优先与 openEuler 独有的架构支持。如升级后可以直接获取相关架构支持，那相关补丁在升级后可以直接 drop；否则架构补丁应随 upstream 升级而刷新。架构补丁的提交者对升级中的刷新工作负责，如果特定架构无法随 upstream 演进，则不适宜作为 openEuler LTS 版本的正式支持对象。

### 打包规范
https://gitee.com/openeuler/community/blob/master/zh/contributors/packaging.md