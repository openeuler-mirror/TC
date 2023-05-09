---
标题:     openEuler LTS 版本内核 KABI 变更策略
类别:     流程设计
摘要:     LTS SP 版本 KABI 变更策略评审
作者:     蒋坤坤(Virt SIG)、孔新伟(Wayca SIG)、曾昭荣(Intel Arch SIG)、谢秀奇(Kernel SIG)
状态:     初始化
编号:     oEEP-0009
创建日期: 2023-05-09
修订日期: 2023-07-27
---

## 动机/问题描述:

内核 KABI 兼容，意味着驱动 （ko，kernel module) 不用重新编译，就可以直接安装使用。openEuler LTS 版本的内核 KABI，保持相对稳定，以便内核更新时，老版本的驱动依然能够使用。但是，KABI 兼容同样也会对内核升级或者支持新硬件、新特性造成比较大的阻碍，使得对核心数据结构、接口变更比较大的特性，很难合入。

上游社区 Linux 6.3 以后，合入 [IOMMUFD Generic interface](https://lwn.net/Articles/912515/)，重构了 iommu 、VFIO 、MDEV 等驱动核心子系统。下一代处理器，包括 x86、arm64 等，都依赖这几个子系统，需要 backport  到openEuler 22.03 LTS SP3 (5.10 内核）。而该框架改动大、复杂度高，难以通过接口适配进行兼容。

因此为了兼顾兼容性和特性演进，希望 openEuler LTS SP 版本间放宽 kABI 限制，以便能更快的支持 Intel EMR 等新一代处理器。

### 新特性简介

加速器驱动 & VSVA
- 在虚拟机端的计算节点使用加速器实现加解密硬件加速：AES256、RSA2048、SM3、SM4
- 加速器驱动基于SVA（Shared Virtual Address）开发，在虚拟机端需支持VSVA技术

- 基于IOMMU Nested mode的VSVA在加速器和CPU之间共享统一地址页表
- 基于IOMMU，限定设备和进程的访问权限和安全边界
- 设备资源不暴露，更安全
- VSVA性能比NO-VSVA性能提升

IOMMUFD/SIOV介绍
- SIOV共享设备资源，具有可扩展性和灵活性
- 更低的硬件成本和复杂性
- 使用不同的抽象层与不同地址域共享

支持SIOV的Intel设备
- DSA(Data Streaming Accelerator)
  - 加速常见数据移动操作，提高依赖于数据移动移动的应用的性能
  - 内核将数据移动操作卸载到DSA，释放CPU周期用于更高优先级的工作
- IAA(In-memory Analytics Accelerator)
  - 提供高吞吐量压缩/解压缩
  - 压缩16Gbps（4个实例），解压缩120Gbps（4个实例），延迟4~10us
- QAT(QuickAssist Technology)
  - 提供卓越能力：400Gb/s加密、160Gpbs压缩、100Kops公钥加密
  - 适用于：存储、HPC&大数据、数据库等
- CVL (Intel 100G NIC)
  - 为云计算工作负载极大减少应用响应时间，提高可预测性，提供高带宽和大吞吐量
  - 适用于web服务，数据库应用、边缘计算服务、缓存服务、存储服务等。

### IOMMUFD 特性引起的 KABI 变更

| vfio 相关的接口                     | **变化原因**             |
| ---------------------------------- | ------------------------ |
| io_add_group_dev                   | 被删除                   |
| vfio_del_group_dev                 | 被删除                   |
| vfio_info_add_capability           | 无变化                   |
| vfio_info_cap_shift                | 无变化                   |
| vfio_pin_pages                     | 传参变化&&传参结构体变化 |
| vfio_register_iommu_driver         | 传参结构体变化           |
| vfio_register_notifier             | 被删除                   |
| vfio_set_irqs_validate_and_prepare | 无变化                   |
| vfio_unpin_pages                   | 传参变化&&传参结构体变化 |
| vfio_unregister_iommu_driver       | 传参结构体变化           |
| vfio_unregister_notifier           | 被删除                   |

| iommu 相关的接口           | **被删除**     |
| ------------------------- | -------------- |
| iommu_get_domain_for_dev  | 传参结构体变化 |
| iommu_group_add_device    | 传参结构体变化 |
| iommu_group_alloc         | 传参结构体变化 |
| iommu_group_get           | 传参结构体变化 |
| iommu_group_id            | 传参结构体变化 |
| iommu_group_put           | 传参结构体变化 |
| iommu_group_remove_device | 传参结构体变化 |
| iommu_iova_to_phys        | 传参结构体变化 |
| iommu_map                 | 传参结构体变化 |
| iommu_unmap               | 传参结构体变化 |

| mdev 相关的接口         | **变化原因**             |
| ---------------------- | ------------------------ |
| mdev_dev               | 传参变化&&传参结构体变化 |
| mdev_from_dev          | 被删除                   |
| mdev_get_drvdata       | 被删除                   |
| mdev_parent_dev        | 被删除                   |
| mdev_register_device   | 被删除                   |
| mdev_register_driver   | 传参变化&&传参结构体变化 |
| mdev_set_drvdata       | 被删除                   |
| mdev_unregister_device | 被删除                   |
| mdev_unregister_driver | 传参结构体变化           |
| mdev_uuid              | 被删除                   |

## 接口变更影响

### 受影响驱动模块

openEuler22.03-LTS(aarch64) 受影响的驱动模块

| 驱动模块名称      | 有影响的函数变化                                             |
| ---------------- | ------------------------------------------------------------ |
| amdgpu           | iommu_get_domain_for_dev iommu_iova_to_phys                  |
| nouveau          | iommu_map iommu_unmap                                        |
| vfio_mdev        | mdev_unregister_driver mdev_register_driver vfio_add_group_dev vfio_del_group_dev |
| vfio-pci         | vfio_add_group_dev vfio_del_group_dev iommu_group_get iommu_group_id iommu_group_put iommu_get_domain_for_dev |
| mdev             | iommu_group_add_device iommu_group_alloc  iommu_group_id iommu_group_put iommu_group_remove_device |
| vfio_iommu_type1 | vfio_register_iommu_driver、vfio_unregister_iommu_driver、iommu_map iommu_unmap、iommu_iova_to_phys |
| vfio             | iommu_group_add_device、iommu_group_alloc、iommu_group_get、iommu_group_id、iommu_group_put、iommu_group_remove_device |
| svm              | iommu_group_get、iommu_group_put                             |
| hisi_sefc2       | iommu_get_domain_for_dev                                     |
| nicvf            | iommu_get_domain_for_dev、iommu_iova_to_phys                 |
| mlx5_core        | mdev_dev、mdev_from_dev、mdev_get_drvdata、mdev_parent_dev、mdev_register_device、mdev_set_drvdata、mdev_unregister_device |

openEuler22.03-LTS(x86_64) 受影响的驱动模块

| 驱动模块名称      | 有影响的函数变化                                             |
| ---------------- | ------------------------------------------------------------ |
| snd-emu10k1      | iommu_get_domain_for_dev                                     |
| usnic_verbs      | iommu_map iommu_unmap                                        |
| iommu_v2         | iommu_group_get iommu_group_put                              |
| nouveau          | iommu_map iommu_unmap                                        |
| amdgpu           | iommu_get_domain_for_dev iommu_iova_to_phys                  |
| vfio_mdev        | mdev_unregister_driver mdev_register_driver vfio_add_group_dev vfio_del_group_dev |
| vfio-pci         | vfio_add_group_dev vfio_del_group_dev iommu_group_get iommu_group_id iommu_group_put iommu_get_domain_for_dev |
| mdev             | iommu_group_add_device iommu_group_alloc  iommu_group_id iommu_group_put iommu_group_remove_device |
| vfio_iommu_type1 | vfio_register_iommu_driver、vfio_unregister_iommu_driver、iommu_map iommu_unmap、iommu_iova_to_phys |
| vfio             | iommu_group_add_device、iommu_group_alloc、iommu_group_get、iommu_group_id、iommu_group_put、iommu_group_remove_device |
| mlx5_core        | mdev_dev、mdev_from_dev、mdev_get_drvdata、mdev_parent_dev、mdev_register_device、mdev_set_drvdata、mdev_unregister_device |

## 业界其他参考

| OS   | SP 内 | SP 间                               |
| ---- | ----- | ----------------------------------- |
| SUSE | 兼容  | 不兼容                              |
| RHEL | 兼容  | RHEL 7、8 白名单兼容; RHEL 9 不兼容 |

* RHEL 9 开始放弃了跨 SP 兼容的方案，通过 kmod 方式为每个 SP 版本准备更多可用的驱动。

https://bugzilla.redhat.com/show_bug.cgi?id=2059183

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/9.0_release_notes/index
Red Hat protects kernel symbols only for minor releases

Red Hat guarantees that a kernel module will continue to load in all future updates within an Extended Update Support (EUS) release, only if you compile the kernel module using protected kernel symbols. There is no kernel Application Binary Interface (ABI) guarantee between minor releases of RHEL 9.

https://sigs.centos.org/kmods/
https://wiki.centos.org/SpecialInterestGroup/Kmods

The Kmods SIG focuses on packaging and maintaining kernel modules for CentOS Stream and Enterprise Linux. We welcome anybody that's interested and willing to do work within the scope of the Kmods SIG to join and contribute. See membership for how to join.

* 以 MLNX_OFED 驱动为例，对各OS发行版部分版本的支持情况

https://network.nvidia.com/products/infiniband-drivers/linux/mlnx_ofed/

| MLNX_OFED   | OS                | OS 版本                                                      |
| ----------- | ----------------- | ------------------------------------------------------------ |
| 5.9-0.5.6.0 | RHEL/CentOS/Rocky | RHEL/Rocky 9.1<br />RHEL/Rocky 9.0<br />RHEL/Rocky 8.7<br />RHEL/Rocky 8.6<br />... |
| 5.9-0.5.6.0 | SUSE/SLES         | SLES 15 SP4<br />SLES 15 SP3<br />SLES 15 SP2<br />SLES 12 SP5<br />SLES 12 SP4<br />SLES 12 SP3<br />... |
|             | KYLIN             | KYLIN 10 SP2<br />KYLIN 10                                   |
|             | UOS               | UOS 20.1040<br />UOS 20.1020                                 |
|             | Ubuntu            | Ubuntu 22.10<br />Ubuntu 22.04<br />Ubuntu 20.04<br />Ubuntu 18.04 |
|             | Debian            | Debian 9.13<br />Debian 11.3<br />Debian 10.9<br />Debian 10.8 |
|             | OPENEULER         | OPENEULER 22.03<br />OPENEULER 20.03 SP3                     |

## 方案及分析

### 原 KABI 策略

openEuler LTS 版本内原则上不改变 KABI，保持兼容，如遇到 KABI 不能兼容的情况，需要在广泛征求意见的基础上，在 TC 进行决策，是否允许变更。

### 方案1

概述：不改变原 KABI 策略，TC 决策，允许 22.03 LTS SP3 允许 KABI 变更，接口总体兼容范围不扩大和缩小，重新生成 kabi 接口校验文件。

方案目的：按需提交变更，变更作为特例，进行广泛评审和征求意见。

存在问题：变更流程较长，节奏不固定，不利于处理器厂商提前安排合入节奏，和驱动厂商出驱动版本的节奏。

方案优点：KABI 稳定优先，严格控制变更。

### 方案2

概述：更新 KABI 策略，允许年度 SP 版本允许 KABI 变更，SP 版本内 KABI 仍保持不变。SP 版本内，白名单兼容范围可以适当扩大，以便兼容更多的驱动。

方案目的：平衡快速演进与生态稳定。

方案详述：

1. SP 版本一旦发布，白名单内的 KABI 维持稳定，除非严重的安全漏洞不得不变更，其他情况维持不变；
2. SP 内的 KABI 白名单适当扩大，保障 update 版本更新，不影响常见的驱动；
3. openEuler 周边驱动围绕 SP 版本进行生态建设，即年度的 SP 版本发布之后，需要驱动重新编译出版本；
4. 与 openEuler 进行兼容性适配的驱动，也应该指明驱动适用于openEuler哪个 SP 版本；

### 方案3

概述：LTS 版本内核，不允许 KABI 变更。如果可以修复 KABI 保持兼容，则允许特性合入，否则不允许合入。

方案目的：严格保持 KABI 在版本周期内兼容，宁愿舍弃部分特性和硬件支持。

### 方案4

概述：openEuler LTS 版本内，严格限制 KABI 变更。下一个 LTS 版本（比如原先规划的 24.03 LTS），即便使用相同的内核（比如 5.10 内核），也允许变更。

方案目的：将 KABI 维持在 LTS 生命周期内（约2年时间），跨 LTS 的版本，允许 KABI 变更。

方案依赖：该方案，依赖 LTS 版本的生命周期和版本节奏。比如如果 24.03 不出基于 5.10 的 LTS 版本，则不能满足前面的要求。

## 影响分析

跨 SP 允许 KABI 变更对各团队的影响

| 团队           | 影响                                                         |
| -------------- | ------------------------------------------------------------ |
| 处理器厂商     | 放宽了跨SP版本的 KABI 限制，处理器新特性可以更容易合入openEuler版本 |
| 驱动厂商       | 驱动厂商需要针对 SP 版本出驱动，增加了版本                   |
| OSV            | 1. 更快拿到新 SP 版本的处理器新特性<br />2. 如果基于openEuler 新 SP 出版本，则南向驱动需要从新编译<br />3. 由于跨 SP 存在 KABI 不兼容的情况，可能影响与openEuler的兼容性认证的范围，比如按 SP 版本进行兼容性认证。 |
| 企业自用版     | 1. 如要支持新硬件，就需要升级 SP 版本，或自行 Backport<br />2. SP 版本升级，可能导致大量驱动不可用，如涉及大量第三方驱动，则导致升级成本骤增。 |
| openEuler 社区 | 1. 有利于延长大版本的支持时间：处理器特性可以更容易合入内核，老内核可以支持更长时间<br />2. 避免 LTS 版本因 KABI 限制导致重要特性缺失<br />3.由于驱动配套频度增加，可能导致部分SP版本驱动配套不足。|

## 讨论及评审
- 经多次技术委员会会议评审，认为下现阶段只决策和推进 22.03 LTS SP3 的 KABI 变更，对于下一个内核大版本周期的 KABI 定义问题，单独进行讨论。
- 会议还邀请了 QLogic 驱动提供商 Mavell 交流驱动发布策略:
 - 每年发两个驱动版本，会跟进openEuler社区最新的SP版本。
 - 当前对openEuler LTS 的KABI白名单没有额外诉求。

## 范围/相关人员及之策
### Kernel SIG
- 梳理 22.03 LTS SP3 KABI 变更列表及原因，提出 ISSUE 和 PR 进行评审。
- 相关参与评审的人员和团队需要包括：兼容性 SIG、OSV、处理器团队、Release SIG 等。

### 兼容性 SIG
- 根据 22.03 LTS SP3 拟变更的 KABI 列表，需要通知到周边影响团队，并听取评审意见，如有必要，提前采取缓解措施。
- 评估对OS兼容性验证的影响，及是否需要修订兼容性标准等。

### Release SIG
- 知会到下游 OSV 及周边相关团队，告知变更范围及影响，如有必要，并提前采取缓解措施。

## 实施计划
openEuler 22.03 LTS SP3 完成变更。
