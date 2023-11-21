---
标题:     openEuler官方虚拟机镜像发布流程
类别:     流程设计
摘要:     官方虚拟机镜像发布流程定义
作者:     鲁卫军 (wjunlu217 at gmail.com)
状态:     活跃
编号:     oEEP-0015
创建日期:  2023-10-24
修订日期:  2023-10-24
---

## 动机/问题描述:
目前发布在openEuler官方网站的虚拟机镜像存在以下问题：
1. 发布镜像预安装的基础软件缺失，如wget, hostname, unzip等，导致用户首次在openEuler上部署应用时经常失败。
2. 发布镜像没有针对云场景进行适配，发布的虚拟机镜像因缺乏关键的cloud-init组件在导入公有云平台时经常失败。
3. 除此以外，没有update发布流程，使得用户无法体验openEuler每个版本的最新特性。

如何决策虚拟机镜像中预安装软件的内容、用户如何按照自身需求定制镜像，以及如何发布更新版本这一系列流程需要被规范。

### 本oEEP解决的问题
- 问题1：虚拟机镜像发布版本如何决策预置软件的内容、用户如何根据发布的镜像进行私有化定制。
- 问题2：当前Release发布的虚拟机镜像，仅在首个版本进行发布，而update版本未进行发布。

## 方案的详细描述:
### 1. 相关SIG组及职责
1. openEuler Gate Keeper SIG：负责官方虚拟机镜像的构建、裁剪，以及构建组件的代码合入，代码仓库为 https://gitee.com/openeuler/openeuler-os-build 和 https://gitee.com/openeuler/CreateImage/
2. openEuler Release SIG：负责发布虚拟机镜像的原始版本及更新版本，由每个版本的Release Manager发布镜像至 https://repo.openeuler.org/；同时，由Release Manager负责每个版本镜像预装软件范围变更的审核
3. openEuler Infra SIG：负责一键发布组件EulerPublisher的设计、开发与维护，基于 https://repo.openeuler.org/ 发布的镜像，向用户提供不同使用场景下镜像预安装软件的定制能力


### 2. 预置软件变更
1. 虚拟机镜像内预安装软件清单rpmlist保存在 https://gitee.com/openeuler/release-management 仓库，形式如下
```
# rpmlist示例
kexec-tools
net-tools
iproute
...

```
2. 镜像预置软件范围需要变更时，通过提交PR来修改rpmlist文件，并由Release Manager审核合入后生效

### 3. 发布流程
由Release SIG发布"openEuler-{VERSION}-{ARCH}.qcow2.xz"至repo.openeuler.org。
- (已有发布) 对于首个版本发布，发布至：
https://repo.openeuler.org/openEuler-{VERSION}/virtual_machine_img/

- (未有发布) 对于update版本发布，发布至：
https://repo.openeuler.org/openEuler-{VERSION}/virtual_machine_img/update/YYYY-MM-DD/
；最新update版本覆盖拷贝至：https://repo.openeuler.org/openEuler-{VERSION}/virtual_machine_img/update/current/。
