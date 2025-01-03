---
标题:     openEuler社区部分软件包自动化管理流程
类别:     流程设计
摘要:     软件包自动化管理
作者:     曹志 (george at openeuler.sh)
状态:     初始化
编号:     oEEP-0019
创建日期: 2024-11-28
修订日期: 2024-11-28 

---

## 动机/目标描述:
### 动机
为提升openEuler社区软件包有效维护效率，增强软件包自动化、智能化管理能力，基础设施协同社区技术委员会和Release SIG组对当前社区管理的所有软件包进行分类筛查，确定某些满足一定特征的软件包在当前技术背景下，适合由社区机器人自动化管理，以减少开发者重复劳动，方便投入更有价值的问题解决和竞争力开发。

### 目标
1. 明确社区哪些软件包在当前技术背景下更适合自动化管理；
2. 将上述软件包归一到EcoPkg SIG 或者新建AutoMaintain SIG组（待确认），独立集中管理；
3. 对上述适合自动化管理的软件包，从包引入到包构建再到后期维护的全流程尽可能自动化；
4. 对软件包自动化管理配套独立文档进行流程讲解和指导；

## 方案详细描述:

### 1. 识别哪些软件包适合自动化管理
根据专家团讨论结论，建议对满足以下条件的软件包，优先落实自动化管理：
1. 社区各大类语言依赖包，如python-，perl-，nodejs-等，他们的共同特点是数量大，人工维护成本高；
2. src-openeuler制品仓中仅包含源码tar包，没有额外的patch，方便自动升级；
3. 软件包自身包含测试功能，即SPEC中含有%check逻辑，以方便自动化校验；

在下文中，暂且将满足上述条件的适合自动化管理的软件包称为自维护包；

### 2. 软件包引入流程适配
对自维护包的引入流程，不再通过[修改community仓库配置文件](https://gitee.com/openeuler/community/blob/master/zh/contributors/create-package.md#%E6%93%8D%E4%BD%9C%E6%AD%A5%E9%AA%A4)的操作方式来完成，
建议使用社区[EUR工具](https://eur.openeuler.openatom.cn/coprs/)和[软件包贡献工具](https://software-pkg.openeuler.org/zh/package)配合完成，具体操作文档见社区博客（待添加）。

软件包引入流程中需要配置软件包完整的上游信息，包括软件包源码仓库和版本维护信息等。

### 3. 自维护包自动化管理方案
社区基础设施将对识别的自维护包进行以下操作：
 1. 自动感知：根据软件包源码仓库信息自动监控上游软件包版本变更；
 2. 自动升级：自动拉取上游新版本进行本地构建验证；
 3. 问题处理：若本地构建验证失败，由机器人提交ISSUE通知失败；
 4. 自动提交：若本地构建验证通过，由机器人提交源码升级PR（含源码包+SPEC文件）；

### 4. 自维护包如何进入版本管理
自维护包如果暂时没有在正式发布的版本中，又希望进入Release正式发布版本，和之前流程一致，可以通过申请PR加入版本，具体操作参见[软件包管理策略原则](https://gitee.com/openeuler/community/blob/master/zh/technical-committee/governance/software-management.md?skip_mobile=true#openeuler-%E8%BD%AF%E4%BB%B6%E5%8C%85%E7%AE%A1%E7%90%86%E7%AD%96%E7%95%A5%E5%8E%9F%E5%88%99)；

### 4. 自维护包责任主体
软件包管理主要分为软件包引入、构建、升级、问题修复等；
| 流程 | 责任主体  |
|--|--|
| 软件包引入 | 软件需求方，社区开发者 |
| 门禁&构建 | 社区机器人 |
| 感知上游版本变更 | 社区机器人 |
| 软件升级 | 社区机器人 |
| 问题修复 | 软件需求方，社区开发者|

## 实施策略
为减小自动升级对社区产生影响：
 1. 变更流程会逐步开展，前期使用一部分软件包试用打样，预计优先试用python-语言依赖包约100个；
 2. 待自动升级全流程技术成熟后，对上述提到的所有语言依赖包（python-，perl-，nodejs-等）开展自动化维护，约2K软件包；
 3. 待2K语言包自动化维护一段时间（约3-6个月）后，再对社区满足条件的其他包全面开展自动化维护；

## 影响评估
该变更主要影响被社区定位为自维护包的这一部分软件包，影响软件包引入流程和维护流程。


