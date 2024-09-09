---
Title:          openEuler Kconfig Option Refactoring and Kernel Package Splitting
Type:           Standard
Abstract:       More module Kconfig options and split kernel packages for on-demand installation
Author:         Liu Kai <kai.liu@xfusion.com>
Status:         Accepted
No:             oEEP-0006
Create_time:    2023-07-20
Update_time:    2023-07-26
---

## Motivation and Objectives

Compared to other mainstream Linux distributions, openEuler has significantly fewer **=m** Kconfig options enabled to build features as modules. As most hardware drivers exist as modules, this difference results in limited hardware driver support, hindering broader hardware platform compatibility.

The following table provides a simplified comparison of Kconfig option usage across mainstream x86_64 Linux distributions.

| Distribution             | Kernel Version| =y         | =m         | Unset   |
| ------------------- | -------- | ---------- | ---------- | ---------- |
| openEuler 22.03 LTS SP2| 5.10     | 2,021 / 30% | 2,362 / 35% | 2,441 / 36% |
| CentOS 8.5          | 4.18     | 1,964 / 30% | 2,534 / 38% | 2,105 / 32% |
| Debian 12           | 6.1      | 2,407 / 28% | 3,861 / 45% | 2,336 / 27% |
| Fedora 38           | 6.3      | 2,679 / 31% | 4,140 / 47% | 1,899 / 22% |
| openSUSE 15.5       | 5.14     | 2,569 / 29% | 4,529 / 52% | 1,633 / 19% |
| Arch Linux 20230717 | 6.4.3    | 2,865 / 30% | 5,747 / 61% | 814 / 9%   |
| Ubuntu 22.04.02     | 5.15     | 2,790 / 30% | 5,837 / 62% | 780 / 8%   |

Due to variations in kernel versions and supported Kconfig options across distributions, it is more reasonable to compare the proportions of each option type. From the table, we can draw the following conclusions:

1. All distributions maintain a similar proportion of the **=y** option, around 30%. However, significant discrepancies exist in **=m** and unset options.
2. Other distributions have higher proportions of the **=m** option compared to openEuler.
3. Desktop-oriented distributions like Arch Linux and Ubuntu exhibit notably higher **=m** option proportions.
4. Distributions catering to both desktop and server use cases, like Debian, Fedora, and openSUSE, also demonstrate considerably higher **=m** option proportions than openEuler, but lower than desktop-oriented distributions.
5. CentOS, primarily server-oriented, still surpasses openEuler in the **=m** option proportion.

Therefore, to bridge the gap with other distributions and support a broader spectrum of hardware devices, particularly PC hardware commonly used by developers and individuals, we need to enable more features to be built as modules.

## Benefits and Risks for openEuler

### Benefits

1. Hardware support is enhanced, reducing user inconvenience caused by kernel recompilation or separate driver installations for unsupported devices.
2. A larger user base can test and utilize openEuler on real hardware, leading to improved bug reporting and southbound compatibility.
3. Splitting the kernel package allows virtualization and cloud deployments to omit unnecessary physical hardware drivers, minimizing drive footprint.

### Risks

1. The kernel package size is significantly increased, impacting scenarios with stringent drive space requirements.
2. Kernel package compilation, packaging, testing, and gated check-in processes will require significantly more time.
3. Code for disabled features may raise errors when compiled and require debugging. Enabling more features will also increase future code maintenance efforts.
4. Theoretically, enabling more features and drivers broadens the attack surface of the kernel, potentially exposing more vulnerabilities.
5. Supporting additional hardware drivers and features might lead to a surge in support and service requests.

To mitigate the risk of increased package size, we propose split kernel packages, allowing users to install only the required modules, minimizing drive space consumption.

## Solution Description

### Kconfig Option Adjustments

- Analyze the driver module policies employed by comprehensive distributions like Fedora, openSUSE, and Debian to establish openEuler's module policy. Aim for an **=m** proportion of around 50%.
- Reclassify previously unset options related to drivers and modular features according to the defined module policy. Options remaining unset due to mutually exclusive selections should remain unchanged.
- Apply the established principles to new Kconfig options introduced by future code backporting or version upgrades, setting eligible options to **=m**.

### Kernel Package Splitting

Enabling more **=m** kernel configuration options necessitates splitting the kernel package into the following subpackages.

| Name          | Content                                                                | Function                                                        |
| -------------- | -------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| kernel-minimal| Essential kernel files, including **vmlinuz** and core .ko files such as those for file systems, iptables, and virtio                                   | Provides the minimal kernel required for booting in virtualized environments, including virtualization and cloud deployments.|
| kernel-base    | Additional .ko files primarily supporting server platform hardware, such as RAID controllers and network interface cards            | Supports common server hardware, enabling out-of-the-box bare metal deployments.                            |
| kernel-extras  | .ko files for PC, embedded, and other platform hardware, such as drivers for Wi-Fi, sound cards, Bluetooth, and multimedia devices.| Supports desktop PC and embedded development devices commonly used by developers and individual users.|

These subpackages have a hierarchical dependency structure. Installing kernel-base, for example, automatically introduces kernel-minimal, ensuring the installation of essential kernel files.

Modifications are required to the existing kernel package spec file to define these subpackages and package files accordingly. Existing subpackages such as kernel-{devel,headers,source,tools,tools-devel} remain unchanged.

### Upgrade Policy

The current kernel package should be migrated to the kernel-base package during system upgrades because:

- The module scope of the kernel-base package closely aligns with the original kernel package.
- The server-oriented kernel-base package is applicable to openEuler's primary deployment scenario.

### Installation Process Enhancements

oemaker needs to be modified to accommodate the new kernel subpackages.

Anaconda used by the installer needs to be enhanced to automatically recommend appropriate kernel subpackages based on detected platform features, while allowing users to manually select desired packages.

## Scope, Personnel, and Responsibilities

### Kernel SIG

- Comprehensive evaluation, policy definition, and adjustment of Kconfig option changes
- Compilation error resolution
- Kernel package spec file modifications
    - Definition of module file lists of each subpackage
- Ongoing maintenance and support for future releases

### OS-Builder SIG

- Anaconda modifications
- oemaker modifications

### QA SIG

- Definition of testing strategies for each spilt kernel subpackage
- Adaptation of kernel testing processes
- Adaptation of installation scenario testing processes

### Compatibility SIG

- Definition of compatibility strategies for each kernel subpackage, encompassing community compatibility testing and OSV certification consistency, to prevent adverse impacts on OSV version certification. For instance, modules within the kernel-extras package might not be subject to mandatory OSV certification compatibility requirements.
- Adaptation of compatibility testing processes

### Infra SIG

- Adaptation of OBS/EBS build environments
- Adaptation of virtual machine image deliverables

### Doc SIG

- Updates to release notes and documentation

## Scope and Affected Software

The following software packages/repositories will be affected and require modification:

- Kernel
- Anaconda
- oemaker

## User Experience

Overall user experience is expected to improve or remain at least the same.

- Developers: enhanced support for development devices
- Individual users: enhanced support for PC peripheral devices
- Physical server users: minimal impact on experience. Enabling previously disabled server hardware drivers might lead to minor improvements.
- Cloud users: slightly improved experience. Installing kernel-minimal can lead to minor drive space savings.

## Release Notes

The release notes should include information about the kernel package splitting, detailing the purpose and use cases of each subpackage.

## Impacts on Upgrade and Compatibility

Appropriate **Provides** and **Obsoletes** settings are crucial to ensure:

- Seamless upgrades from older versions with a single kernel package, eliminating the need for manual kernel subpackage installations during initial upgrades
- Automatic dependency resolution for other software packages relying on the kernel package

## How to Test

Due to the extensive hardware coverage of newly enabled kernel module compilation, including numerous non-server hardware, comprehensive functional and compatibility testing is impractical. Therefore, the strategy is as follows:

- Maintain existing community testing coverage for devices, drivers, and functionalities, ensuring no regression in experience. Adapt test environments to the new subpackage names to prevent omissions in testing due to package name changes.

- Perform minimal testing on newly added kernel modules (primarily in the kernel-extras package) to ensure RPM package integrity, proper installation/uninstallation processes with initramfs updates, and no adverse effects on system booting.

## Implementation Plan

The plan is to implement and validate these changes in openEuler 23.09 (innovation version) and incorporate them into openEuler 24.03 LTS.
