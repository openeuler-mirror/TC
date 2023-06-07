---
标题:     通过 oEEP ( openEuler 演进提案 Evolution Proposal) 规范化社区演进的技术决策流程
类别:     流程设计
摘要:     为 openEuler 增加 oEEP 流程
作者:     胡欣蔚 (shinwell_hu at openeuler.sh)
状态:     活跃
编号:     oEEP-0001
创建日期: 2023-04-16
修订日期: 2023-05-25
---

## 动机/问题描述:
  openEuler 社区经过了3年的演进和发展，过程中已经发生了很多关键的技术决策。这些技术决策的过程分散在各次技术委员会、SIG的会议当中；这些技术决策的传递和落实，通过开发者之间的直接沟通；这些技术决策有时候会通过openEuler公众号公开，有时候会通过视频讲解，但更多时候会忽略了对社区的推广。

  这使得我们对 openEuler 技术演进的追溯过于依赖开发过程的亲历者，社区的技术决策欠缺透明性，难以聚集更大规模的社区开发人员参与。

  参考其他成熟的社区，比如Python, OpenStack, Debian等，都通过社区定义的一系列文档来承载技术演进决策，这些文档公开可查，可参与讨论，使得社区开发人员对技术发展的脉络，将来的走向，都可以有充分的理解。

  openEuler 需要学习这些成功的经验，定义一套适合 openEuler 的文档与流程规范。

## 方案的详细描述：

### oEEP 的目标读者
 oEEP 目标读者是 openEuler 社区当前和未来的开发者，实践者和技术决策者。oEEP 不是技术推广、宣传文档。

### oEEP 的类型

  oEEP 可以分为三个类型：
- **特性变更**：描述通过openEuler 的代码、工具、配置变更，提供新的功能或实现的变更。这些变更对开发者、系统管理员、用户都会产生可见的显著特性变化。需要 openEuler 社区的审查和讨论。
- **信息整理**：提供有关 openEuler 社区的信息，如社区索引，指南，规范，开发流程、环境或实践。
- **流程设计**：改进或调整社区治理、开发过程、基础设施等方面的提案。

### oEEP 开发流程

  所有的 oEEP 都从 **初始化** 状态开始。但涉及的开发流程，对于 **特性变更**，**信息整理**，**流程设计** 三类，并不完全一样。

  对于 **特性变更** 类 oEEP，从 **初始化** 状态开始，首先需要确认受影响的开发者和用户，通过各渠道讨论，在受影响范围内形成基本共识，这时的 oEEP 被认为是 **基本成型（Provision）** 的状态。
  处于 **基本成型** 状态的 oEEP 需要通过技术委员会的正式决策，这过程中，技术委员会可以进一步挑战提案的影响范围分析是否完整，落实步骤是否可行。当通过技术委员会评审决策通过后，oEEP 进入 **接纳** 状态；
  反之，oEEP 会停留在 **基本成型** 状态继续修改，或者进入 **拒绝** 状态。进入 **接纳** 状态的 oEEP不再接受内容大幅度修订，一旦落实完成，则变为 **已完成** 状态。
  如果 **接纳** 状态的 oEEP 长期无法完成，技术委员会可以将状态退回 **基本成型** 状态，重新发起评审决策。如果 oEEP 在  **初始化** 或 **基本成型** 状态中，
  但超过 6 个月没有任何进展，技术委员会将该 oEEP 标记为 **不活跃** 状态，一旦后续重启讨论，oEEP 还需要重新从 **初始化** 状态开始。在整个过程中，提案人在任何时候都可以主动撤回提案，这时 oEEP 进入 **撤回** 状态，不再做任何修改，但依然归档。

  **流程设计** 是一种特殊的 **特性变更** oEEP，影响范围主要是社区基础设施与技术委员会自身。**流程设计** 的开发流程和 **特性变更** 类 oEEP 一致。

  **信息整理** 类 oEEP 在初始化之后，经过作者与相关方讨论和迭代，进入 **基本成型** 状态。**基本成型** 的 **信息整理** 类 oEEP，通过技术委员会委员审读通过，进入 **活跃** 状态。
  此时技术委员会授权该 oEEP 作者长期维护，可以根据社区进展持续修订 oEEP 内容。oEEP 作者对 **活跃** 状态的 oEEP 中信息有效性负责。
  如果作者已经无法有效维护该 oEEP，技术委员会负责将该 oEEP 状态改为 **不活跃**，此时 oEEP 中的内容不能保证有效，但可以作为历史信息参考。

  任何一个 oEEP 都可能在社区演进过程中，被后续的 oEEP 取代，此时 oEEP 状态为 **被替代**。 **被替代** 的 oEEP 编号不受影响。

  以下用流程图表示 oEEP 的开发流程

  ```mermaid
  graph LR;
  A(初始化) -- 社区讨论形成基本共识 ---> B[基本成型];
  B[基本成型] -- 提交技术委员会例会 ---> G{技术委员会评审决策?};
  G{技术委员会评审决策?} -- 通过 ---> H{是否为信息整理类oEEP?};
  H{是否为信息整理类oEEP?} -- 是 ---> I(活跃);
  H{是否为信息整理类oEEP?} -- 否 ---> C[接纳];
  G{技术委员会评审决策?} -- 未通过 ---> B[基本成型];
  G{技术委员会评审决策?} -- 反对 ---> D(拒绝);
  C[接纳] -- 落实完成 ---> E(已完成);
  C[接纳] -- 长期无法完成 ---> B[基本成型];
  A(初始化) -- 超过6个月没有进展 ---> F[不活跃];
  B[基本成型] -- 超过6个月没有进展 ---> F[不活跃];
  F[不活跃] -- 重启讨论 ---> A(初始化);
  I(活跃) -- 无法有效维护 ---> F[不活跃];
  ```

### oEEP 格式要求

  提交的oEEP文件名应当以 oEEP-XXXX开头，XXXX 为当前未被占用的最小四位数字编号。
  oEEP 应以 Markdown 格式写作。

### oEEP 的开发

  oEEP 统一以 PR 形式提交到 gitee.com/openeuler/TC/oEEP 目录中。
  处于 **初始化**，**基本成型** 状态的 oEEP 文件可以被反复大规模修订；处于 **接纳**，**活跃**，**完成** 状态的 oEEP 文件可以接受定期，小规模修订；处于**撤回**，**拒绝** 状态的 oEEP 不接受修订。

## 参考文档与链接:
- openstack blueprint说明: [https://wiki.openstack.org/wiki/Blueprints](https://wiki.openstack.org/wiki/Blueprints)
- python PEP说明:[https://peps.python.org/pep-0000/](https://peps.python.org/pep-0000/)
- Fedora changes说明: [https://docs.fedoraproject.org/en-US/program_management/changes_policy/](https://docs.fedoraproject.org/en-US/program_management/changes_policy/)
- Fedora changes的一个案例: [https://fedoraproject.org/wiki/LTOByDefault](https://fedoraproject.org/wiki/LTOByDefault)
- OpenSUSE proposals的说明: [https://en.opensuse.org/openSUSE:Strategy_proposal_overview](https://en.opensuse.org/openSUSE:Strategy_proposal_overview)
- Debian proposals的说明: [https://dep-team.pages.debian.net/](https://dep-team.pages.debian.net/)
