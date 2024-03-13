---
标题:     oEEP (openEuler 演进提案 Evolution Proposal) 索引
类别:     信息整理
摘要:     当前 oEEP 全集索引
作者:     胡欣蔚 (shinwell_hu at openeuler.sh)
状态:     活跃
编号:     oEEP-0000
创建日期: 2023-04-16
修订日期: 2023-07-31
---

## 索引:

| 编号 | 类型，状态 | 标题 | 作者 | 提案日期 |
| :----: | :-----------: | :----: | :----: | :---------: |
| 0001 | P,I | [通过 oEEP 规范化社区演进的技术决策流程](oEEP-0001%20通过%20oEEP%20规范化社区演进的技术决策流程.md) | 胡欣蔚 shinwell_hu at openeuler.sh | 2023-04-13 |
| 0002 | D,I | [oEEP 格式与内容规范](oEEP-0002%20oEEP%20格式与内容规范.md) | 胡欣蔚 shinwell_hu at openeuler.sh | 2023-04-13 |
| 0003 | S,I | [LLVM平行宇宙计划--基于LLVM技术栈构建oE软件包](oEEP-0003%20LLVM平行宇宙计划--基于LLVM技术栈构建oE软件包.md) |赵川峰 cf-zhao | 2023-05-04 |
| 0004 | S,A | [dnf根据用户情况自动优化下载软件包](oEEP-0004%20dnf根据用户情况自动优化下载软件包.md) |陈曾 zengchen1024 at chenzeng2@huawei.com | 2023-06-07 |
| 0005 | P,I | [openEuler官方容器镜像发布流程](oEEP-0005%20openEuler官方容器镜像发布流程.md) | 姜逸坤 yikunkero at gmail.com, 鲁卫军 wjunlu217 at gmail.com | 2023-05-10 |
| 0006 | S,A | [openEuler内核kconfig翻新及内核包拆分](oEEP-0006%20openEuler内核kconfig翻新及内核包拆分.md) | 刘恺 kai.liu at xfusion.com | 2023-07-20 |
| 0007 | D,I | [openEuler优秀项目评选规则](oEEP-0007%20openEuler优秀项目评选规则.md) | 胡欣蔚 shinwell_hu at openeuler.sh | 2023-07-10 |
| 0008 | P,I | [openEuler智能补丁管理框架](oEEP-0008%20openEuler智能补丁管理框架.md) | 胡峰 solarhu| 2023-07-25 |
| 0009 | P,I | [openEuler LTS 版本内核 KABI 变更策略](oEEP-0009%20openEuler%20LTS%20版本内核%20KABI%20变更策略.md) | 谢秀奇 xiexiuqi at huawei.com | 2023-5-9 |
| 0010 | D,I | [openEuler服务要求参考基线](oEEP-0010%20openEuler服务要求参考基线.md) | 邓晖龙 denghuilong88 at huawei.com | 2023-8-2 |
| 0011 | S,I | [openEuler多版本内核支持](oEEP-0011%20openEuler多版本内核支持.md) | 刘恺 kai.liu at xfusion.com | 2023-08-09 |
| 0012 | P,I | [openEuler软件包非upstream支持多架构代码提交规则](oEEP-0012%20openEuler软件包非upstream支持多架构代码提交规则.md) | 叶青龙 yeqinglong@kylinsec.com.cn,胡欣蔚 shinwell_hu at openeuler.sh | 2023-08-17 |
| 0013 | P,I | [openEuler SIG 组孵化流程优化](oEEP-0013%20openEuler%20SIG%20组孵化流程优化.md) | 熊伟 (xiongwei888 at huawei dot com) 胡欣蔚 (shinwell_hu at openeuler dot sh)
| 0014 | P,I | [openEuler AI容器镜像软件栈规范](oEEP-0014%20openEuler%20AI容器镜像软件栈规范.md) | 鲁卫军（wjunlu217 at gmail.com）| 2023-10-23 |
| 0015 | P,I | [openEuler官方虚拟机镜像发布流程](oEEP-0015%20openEuler官方虚拟机镜像发布流程.md) | 鲁卫军（wjunlu217 at gmail.com）| 2023-10-24 |
| 0016 | P,I | [openEuler新增branch-keeper角色作为仓库的分支守护者](oEEP-0016%20openEuler新增branch-keeper角色作为仓库的分支守护者.md) | 曹志（george at openeuler.sh）| 2024-02-08 |

## oEEP 类型分类：
- D (Document, 信息整理): 信息梳理形成的文档。此类 oEPP 包含社区索引，指南，规范或其他和 openEuler 相关的信息。
- P (Process, 流程设计) -- openEuler 社区的流程设计，包括社区治理，CI/CD，测试等相关要求。
- S (Standard, 特性变更) -- openEuler 的代码、工具、配置变更设计，这些设计产生对开发者、系统管理员、用户可见的特性变化。

## oEEP 状态分类：
- I (Initial, 初始化): 当前活跃的 DRAFT，但包含的信息尚未达成共识
- A (Accepted, 接纳): 已被 技术委员会和执行方接受，尚在落实过程中 (对于P, S)
- A (Active, 活跃): 活跃维护，有效的信息 (对于D)
- D (Deactive, 不活跃): 不活跃的 oEEP ，但后续可能重新启动
- F (Final, 已完成): 已完成的 oEEP，不再需要跟进 (对于P, S)
- P (Provision, 基本成型): 基本形成共识，待技术委员会正式决策
- R (Rejected, 被拒绝): 已被技术委员会正式拒绝，不接纳
- S (Substiuted，被替代): 被后续其他的oEEP替代
- W (Withdraw, 撤回): 被提案人撤回
