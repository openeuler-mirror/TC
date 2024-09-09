---
Title:          kABI Change Policy for openEuler LTS Versions
Type:           Process
Abstract:       Review of kABI change policy for LTS SP versions
Author:         Jiang Kunkun (Virt SIG), Kong Xinwei (Wayca SIG), Zeng Zhaorong (Intel Arch SIG), Xie Xiuqi (Kernel SIG)
Status:         Initial
No:             oEEP-0009
Create_time:    2023-05-09
Update_time:    2023-07-27
---

## Motivation/Problem

The kernel Application Binary Interface (kABI) compatibility ensures that drivers in the form of kernel modules (.ko files) can be installed and used without recompilation. openEuler LTS versions aim for stable kABIs to maintain compatibility with older drivers across kernel updates. However, strict kABI adherence can hinder kernel upgrades and the adoption of new hardware and features, particularly those requiring significant changes to core data structures and interfaces.

Upstream Linux 6.3 introduced the [IOMMUFD Generic Interface](https://lwn.net/Articles/912515/), substantially refactoring the IOMMU, VFIO, and MDEV subsystems. Next-generation processors, including x86 and AArch64 architectures, rely on these subsystems, necessitating backporting IOMMUFD to openEuler 22.03 LTS SP3 (kernel 5.10). However, the extensive nature of these changes makes maintaining kABI compatibility challenging.

Therefore, to balance compatibility with feature evolution, we propose loosing kABI restrictions between openEuler LTS SP versions. This will enable faster support for next-generation processors like Intel EMR.

### New Feature Overview

Accelerator driver and virtual Shared Virtual Addressing (vSVA)

- Employs accelerators for hardware-accelerated encryption and decryption within VM compute nodes, supporting AES256, RSA2048, SM3, and SM4.
- Relies on SVA, requiring vSVA support on the VM side.

- Employs IOMMU Nested mode for a unified address space between the accelerator and CPU.
- Enhances security by restricting device and process access permissions.
- Improves security by shielding device resources.
- Delivers performance improvements compared to non-VSVA implementations.

IOMMUFD/SIOV

- Enables resource sharing and scalability through Scalable I/O Virtualization (SIOV).
- Offers reduced hardware complexity and cost.
- Facilitates sharing across different address domains using abstraction layers.

Intel devices supporting SIOV

- Data Streaming Accelerator (DSA)
    - Offloads and accelerates data movement operations for improved application performance, releasing CPU cycles for higher-priority loads.
- In-memory Analytics Accelerator (IAA)
    - Provides high-throughput compression/decompression capabilities.
    - Delivers 16 Gbit/s compression (4 instances) and 120 Gbit/s decompression (4 instances), with a latency of 4 to 10 μs.
- QuickAssist Technology (QAT)
    - Delivers exceptional performance, achieving up to 400 Gbit/s encryption, 160 G/bits compression, and 100,000 operations per second for public key encryption.
    - Ideal for accelerating applications in storage, high-performance computing (HPC), and databases.
- CVL (Intel 100G NIC)
    - Significantly reduces application response time and improves predictability for cloud workloads by providing high bandwidth and throughput.
    - Beneficial for web services, database applications, edge computing services, caching services, and storage services.

### kABI Changes due to IOMMUFD

| VFIO-related Interface                    | Change            |
| ---------------------------------- | ------------------------ |
| io_add_group_dev                   | Removed                  |
| vfio_del_group_dev                 | Removed                  |
| vfio_info_add_capability           | Not changed                  |
| vfio_info_cap_shift                | Not changed                  |
| vfio_pin_pages                     | Modified parameters and structures|
| vfio_register_iommu_driver         | Modified structures          |
| vfio_register_notifier             | Removed                  |
| vfio_set_irqs_validate_and_prepare | Not changed                  |
| vfio_unpin_pages                   | Modified parameters and structures|
| vfio_unregister_iommu_driver       | Modified structures          |
| vfio_unregister_notifier           | Removed                  |

| IOMMU-related Interface          | Change    |
| ------------------------- | -------------- |
| iommu_get_domain_for_dev  | Modified structures|
| iommu_group_add_device    | Modified structures|
| iommu_group_alloc         | Modified structures|
| iommu_group_get           | Modified structures|
| iommu_group_id            | Modified structures|
| iommu_group_put           | Modified structures|
| iommu_group_remove_device | Modified structures|
| iommu_iova_to_phys        | Modified structures|
| iommu_map                 | Modified structures|
| iommu_unmap               | Modified structures|

| MDEV-related Interface        | Change            |
| ---------------------- | ------------------------ |
| mdev_dev               | Modified parameters and structures|
| mdev_from_dev          | Removed                  |
| mdev_get_drvdata       | Removed                  |
| mdev_parent_dev        | Removed                  |
| mdev_register_device   | Removed                  |
| mdev_register_driver   | Modified parameters and structures|
| mdev_set_drvdata       | Removed                  |
| mdev_unregister_device | Removed                  |
| mdev_unregister_driver | Modified structures          |
| mdev_uuid              | Removed                  |

## Impact of Interface Changes

### Affected Driver Modules

Affected driver modules in openEuler 22.03 LTS (AArch64)

| Name     | Affecting Functions                                            |
| ---------------- | ------------------------------------------------------------ |
| amdgpu           | iommu_get_domain_for_dev, iommu_iova_to_phys                 |
| nouveau          | iommu_map, iommu_unmap                                        |
| vfio_mdev        | mdev_unregister_driver, mdev_register_driver, vfio_add_group_dev, vfio_del_group_dev|
| vfio-pci         | vfio_add_group_dev, vfio_del_group_dev, iommu_group_get, iommu_group_id, iommu_group_put, iommu_get_domain_for_dev|
| mdev             | iommu_group_add_device, iommu_group_alloc, iommu_group_id, iommu_group_put, iommu_group_remove_device|
| vfio_iommu_type1 | vfio_register_iommu_driver, vfio_unregister_iommu_driver, iommu_map iommu_unmap, iommu_iova_to_phys|
| vfio             | iommu_group_add_device, iommu_group_alloc, iommu_group_get, iommu_group_id, iommu_group_put, iommu_group_remove_device|
| svm              | iommu_group_get, iommu_group_put                            |
| hisi_sefc2       | iommu_get_domain_for_dev                                     |
| nicvf            | iommu_get_domain_for_dev, iommu_iova_to_phys                |
| mlx5_core        | mdev_dev, mdev_from_dev, mdev_get_drvdata, mdev_parent_dev, mdev_register_device, mdev_set_drvdata, mdev_unregister_device|

Affected driver modules in openEuler 22.03 LTS (x86_64)

| Name     | Affecting Functions                                            |
| ---------------- | ------------------------------------------------------------ |
| snd-emu10k1      | iommu_get_domain_for_dev                                     |
| usnic_verbs      | iommu_map, iommu_unmap                                       |
| iommu_v2         | iommu_group_get, iommu_group_put                             |
| nouveau          | iommu_map, iommu_unmap                                       |
| amdgpu           | iommu_get_domain_for_dev, iommu_iova_to_phys                 |
| vfio_mdev        | mdev_unregister_driver, mdev_register_driver, vfio_add_group_dev, vfio_del_group_dev|
| vfio-pci         | vfio_add_group_dev, vfio_del_group_dev, iommu_group_get, iommu_group_id, iommu_group_put, iommu_get_domain_for_dev|
| mdev             | iommu_group_add_device, iommu_group_alloc, iommu_group_id, iommu_group_put, iommu_group_remove_device|
| vfio_iommu_type1 | vfio_register_iommu_driver, vfio_unregister_iommu_driver, iommu_map, iommu_unmap, iommu_iova_to_phys|
| vfio             | iommu_group_add_device, iommu_group_alloc, iommu_group_get, iommu_group_id, iommu_group_put, iommu_group_remove_device|
| mlx5_core        | mdev_dev, mdev_from_dev, mdev_get_drvdata, mdev_parent_dev, mdev_register_device, mdev_set_drvdata, mdev_unregister_device|

## Industry Practices

| OS   | Within an SP | Across SPs                                                          |
| ---- | ------------ | ------------------------------------------------------------------- |
| SUSE | Compatible   | Not compatible                                                      |
| RHEL | Compatible   | Compatible for RHEL 7 and 8 allowlists<br>Not compatible for RHEL 9 |

- RHEL 9 uses `kmod` to provide drivers for each SP.

<https://bugzilla.redhat.com/show_bug.cgi?id=2059183>

<https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/9.0_release_notes/index>
Red Hat protects kernel symbols only for minor releases

Red Hat guarantees that a kernel module will continue to load in all future updates within an Extended Update Support (EUS) release, only if you compile the kernel module using protected kernel symbols. There is no kernel Application Binary Interface (ABI) guarantee between minor releases of RHEL 9.

<https://sigs.centos.org/kmods/>
<https://wiki.centos.org/SpecialInterestGroup/Kmods>

The Kmods SIG focuses on packaging and maintaining kernel modules for CentOS Stream and Enterprise Linux. We welcome anybody that's interested and willing to do work within the scope of the Kmods SIG to join and contribute. See membership for how to join.

- Taking the MLNX_OFED driver as an example, the support status for some OS versions is as follows:

<https://network.nvidia.com/products/infiniband-drivers/linux/mlnx_ofed/>

| MLNX_OFED   | OS                | Version                                                     |
| ----------- | ----------------- | ------------------------------------------------------------ |
| 5.9-0.5.6.0 | RHEL/CentOS/Rocky Linux | RHEL/Rocky Linux 9.1<br>RHEL/Rocky Linux 9.0<br>RHEL/Rocky Linux 8.7<br>RHEL/Rocky Linux 8.6<br>... |
| 5.9-0.5.6.0 | SUSE/SLES         | SLES 15 SP4<br>SLES 15 SP3<br>SLES 15 SP2<br>SLES 12 SP5<br>SLES 12 SP4<br>SLES 12 SP3<br>... |
|             | Kylin             | Kylin 10 SP2<br>Kylin 10                                   |
|             | UOS               | UOS 20.1040<br>UOS 20.1020                                 |
|             | Ubuntu            | Ubuntu 22.10<br>Ubuntu 22.04<br>Ubuntu 20.04<br>Ubuntu 18.04 |
|             | Debian            | Debian 9.13<br>Debian 11.3<br>Debian 10.9<br>Debian 10.8 |
|             | openEuler         | openEuler 22.03<br>openEuler 20.03 SP3                     |

## Proposed Solutions

### Original kABI Policy

openEuler LTS versions prioritize kABI stability throughout their lifecycle to ensure compatibility. While kABI changes are generally avoided, exceptions may arise. In such cases, the Technical Committee (TC) will conduct thorough consultations and make informed decisions regarding any potential kABI modifications.

### Solution 1

Overview: Do not change the original kABI policy. The TC decides to allow kABI changes for openEuler 22.03 LTS SP3. The overall interface compatibility scope will not be expanded or reduced, and the kABI verification file will be regenerated.

Purpose: Changes can be submitted on demand and treated as special cases for extensive review and consultation.

Issues: The unpredictable pace and length of the change process make it difficult for processor vendors to set integration schedules and for driver vendors to release drivers in a timely manner.

Advantages: kABI stability is prioritized, and changes are strictly controlled.

### Solution 2

Overview: Update the kABI policy to allow kABI changes for annual SP versions, while maintaining kABI stability within SP versions. The scope of the compatibility allowlist can be appropriately expanded within SP versions to accommodate more drivers.

Purpose: To balance rapid evolution with ecosystem stability.

Detailed description:

1. Once an SP version is released, the kABI within the allowlist will remain stable and unchanged unless there are serious security vulnerabilities.
2. The kABI allowlist within the SP will be appropriately expanded to ensure that updates within the version do not affect common drivers.
3. The openEuler peripheral drivers will be built around the SP version, which means that after the annual SP version is released, the drivers need to be recompiled and released.
4. Drivers that are compatible with openEuler should also specify which SP version of openEuler the driver applies to.

### Scheme 3

Overview: No kABI changes are allowed for kernels of LTS versions. Feature integration is allowed only if compatibility can be maintained with fixes.

Purpose: To strictly maintain kABI compatibility throughout the version lifecycle, even at the expense of some features and hardware support.

### Solution 4

Overview: Strictly restrict kABI changes within openEuler LTS versions. Allow changes for the next LTS version (such as the originally planned 24.03 LTS), even if it uses the same kernel (such as 5.10 kernel).

Purpose: To maintain kABI compatibility throughout the LTS lifecycle (approximately 2 years), while allowing kABI changes across LTS versions.

Dependencies: This solution depends on the LTS lifecycle and release cadence. For example, if there is no 5.10-based LTS version for 24.03, the above requirements cannot be met.

## Impacts

Impacts of allowing kABI changes across SPs on different development teams are as follows.

| Team          | Impact                                                        |
| -------------- | ------------------------------------------------------------ |
| Processor vendor    | Relaxed kABI restrictions across SP versions make it easier to integrate new processor features into openEuler.|
| Driver vendors      | Driver vendors need to release drivers for each SP version, increasing the version development effort.                  |
| OSVs           | 1. OSVs have faster access to new processor features in SP versions.<br>2. If a new release is based on the new openEuler SP, hardware drivers need to be recompiled.<br>3. Due to kABI incompatibility across SPs, the scope of compatibility certification with openEuler may be affected, such as performing compatibility certification by SP version.|
| Internal enterprise teams    | 1. If new hardware is to be supported, the SP version needs to be upgraded or the driver needs to be backported.<br>2. SP version upgrades risk driver incompatibility, potentially affecting many third-party drivers and significantly increasing upgrade costs.|
| openEuler community| 1. Processor features are now more easily integrated into the kernel, enabling longer support periods for major versions, including older ones.<br>2. The lack of important features in LTS versions due to kABI restrictions can be avoided.<br>3. Due to the increased frequency of driver adaptation, some SP versions may have insufficient driver support.|

## Discussion and Review

- Following multiple TC reviews, the consensus is to focus on kABI changes for 22.03 LTS SP3. Future kABI policies for subsequent major kernel versions will be discussed separately.
- Feedback from driver provider Marvell indicates that they release two driver versions annually, aligning with openEuler SP releases and currently have no additional requests regarding openEuler's kABI allowlists.

## Scope, Personnel, and Responsibilities

### Kernel SIG

- Identify and document all 22.03 LTS SP3 kABI changes, including reasons.
- Initiate reviews through issues and PRs, involving the Compatibility SIG, OSVs, processor teams, and the Release Management SIG.

### Compatibility SIG

- Communicate kABI changes to impacted teams and gather feedback. Proactively implement mitigation measures as needed.
- Assess the impact on OS compatibility verification and revise compatibility standards accordingly.

### Release SIG

- Inform downstream OSVs and related teams about the changes and potential impact. Proactively implement mitigation measures as needed.

## Implementation Plan

The proposed kABI changes will be implemented in openEuler 22.03 LTS SP3.
