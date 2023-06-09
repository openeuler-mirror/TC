- Title:    oEEP (openEuler 演进提案 Evolution Proposal) 索引
- oEEP:     0000
- State:    活跃
- Date:     2023/05/22
- Driver:   openEuler 技术委员会 (tc at openeuler.org)
- Type:     信息整理
- Abstract: 当前 oEEP 全集索引

## 索引:

| 编号 | 类型，状态 | 标题 | 作者 | 提案日期 |
| :----: | :-----------: | :----: | :----: | :---------: |
| 0001 | P,I | [通过 oEEP 规范化社区演进的技术决策流程](oEEP-0001%20通过%20oEEP%20规范化社区演进的技术决策流程.md) | 胡欣蔚 shinwell_hu at opensuler.sh | 2023-04-13 |
| 0002 | D,I | [oEEP 格式与内容规范](oEEP-0002%20oEEP%20格式与内容规范.md) | 胡欣蔚 shinwell_hu at openeuler.sh | 2023-04-13 |
| 0003 | S,I | [LLVM平行宇宙计划--基于LLVM技术栈构建oE软件包](oEEP-0003%20LLVM平行宇宙计划--基于LLVM技术栈构建oE软件包.md) |赵川峰 cf-zhao | 2023-05-04 |
| 0004 | S,A | [dnf根据用户情况自动优化下载软件包](oEEP-0004%20dnf根据用户情况自动优化下载软件包.md) |陈曾 zengchen1024 at chenzeng2@huawei.com | 2023-06-07 |

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
