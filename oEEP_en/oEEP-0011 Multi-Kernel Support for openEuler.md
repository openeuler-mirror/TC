---
Title:          Multi-Kernel Support for openEuler
Type:           Standard
Abstract:       Multi-kernel support for openEuler LTS versions
Author:         Liu Kai <kai.liu@xfusion.com>
Status:         Initial
No:             oEEP-0011
Create_time:    2023-08-09
Update_time:    2023-08-15
---

## Motivation and Objectives

Currently, openEuler uses a one-to-one binding between the kernel version and the distribution version, fixing the kernel version upon release. This presents several challenges:

1. Determining the LTS version timing is complex, requiring consideration of the LTS version schedule, upstream kernel timelines, new hardware support, and partner/OSV product plans. This often leads to compromises in kernel selection.

2. The extended lifecycle of LTS versions means the kernel becomes relatively outdated by the later stages, hindering support for newer hardware and features. While backporting is possible, it becomes increasingly difficult with widening version gaps.

3. Users may need older LTS versions for application compatibility but desire newer kernels for hardware support.

Decoupling the kernel and user-space versions (adopting an _N_-to-one model) offers a solution. The following are examples of how other distributions address this.

### Ubuntu

The [hardware enablement stacks (HWEs)](https://wiki.ubuntu.com/Kernel/LTSEnablementStack) of Ubuntu provide newer kernels, X, and Mesa packages for LTS versions to support new hardware. Each LTS version receives kernel packages from the next four Ubuntu versions. For example, Ubuntu 20.04 LTS (kernel 5.4) received HWE kernels from 20.10 (5.8) to 22.04 LTS (5.14). Similarly, Ubuntu 22.04 LTS (kernel 5.15) upgraded the kernel to 5.19 (from 22.10) in its 22.04.02 update. In addition, Ubuntu defaults to HWE kernels in minor LTS updates, improving the experience on new hardware.

### Anolis OS 23

This release highlights "dual kernel" support as a key feature, offering installations with both kernel 5.10 (default) and 6.1 (tech preview with rolling updates).

### RHEL 7/CentOS 7

As a counterexample, Red Hat lacks an official kernel upgrade mechanism. The aging 3.10 kernel in RHEL 7/CentOS 7, despite continuous backporting, limits new hardware and container support. Users often resort to [kernel-ml](http://elrepo.org/tiki/kernel-ml) or [kernel-lt](http://elrepo.org/tiki/kernel-lt) packages from the ELRepo community for mainline or LTS kernels.

### openSUSE/SLES

SUSE takes a different approach. While not allowing simultaneous installations of multiple major kernel versions, each Service Pack (SP) upgrade includes a newer kernel. For instance, the kernel of SLES 15 transitioned from 4.12 (GA, SP1) to 5.3 (SP2, SP3), and then to 5.14 (SP4, SP5). This approach balances the backporting strategy of Red Hat with the continuous updates of Ubuntu. However, it lacks the option to retain older kernels during SP upgrades, potentially disrupting users due to out-of-tree module incompatibilities every two years.

This proposal recommends adopting a model similar to that of Ubuntu for openEuler LTS versions, supporting multiple kernel versions to address the current limitations.

## Benefits and Risks for openEuler

### Benefits

- Multi-kernel support eases decision-making for LTS kernel versions.
- It also allows openEuler LTS versions to leverage new hardware and features through newer kernels while maintaining compatibility with upper-layer applications.

### Risks

- Supporting multiple kernel versions increases development and maintenance efforts.

To mitigate this, we suggest offering newer kernels from subsequent openEuler LTS versions on older LTS versions. This minimizes the number of actively developed kernels, focusing efforts on building, packaging, and testing for different user space versions.

## Solution Description

Assume that V1, V2...V*n* represent openEuler LTS versions, and K1, K2...K*n* represent kernel versions.

### Release and Maintenance Strategy

- V1 release: includes K1 as the native kernel.
- V2 release: includes K2 as the native kernel and provides K2 for V1.
- V3 release: includes K3 as the native kernel and provides K3 for both V1 and V2.
    - V1 now supports K1, K2, and K3.
    - V2 now supports K2 and K3.
- With approximately a two-year interval between V1, V2, and V3, this approach ensures:
    - V1 receives two kernel updates (K2, K3), providing up-to-date hardware and feature support for four years and good compatibility for six years.
    - Optionally, V1 can receive K4 or even later kernels if needed.

Kernels introduced to LTS versions beyond their native kernel will be referred to as "LTS upgrade kernels."

### Support Strategy

Several options are available for consideration.

#### Fixed Strategy 1: Fixed Support Period for Each Kernel

- Each K*n* receives a fixed support period (for example, 3 years). Users must upgrade to K*n*+1 or later after K*n* reaches its end of support (EOS).
- Logic: Decouples the OS and kernel lifecycles.
- Pros: shorter support cycles, reduced maintenance burden, faster evolution
- Cons: requirement for users to upgrade kernels actively

#### Fixed Strategy 2: Support Throughout LTS Lifecycle

- On V*n*, every supported kernel (K*n*) is supported until the EOS of V*n*.
- Logic: Ensures all K*n* versions are available throughout the lifespan of V*n*.
- Pros: user-friendliness that allows choosing a suitable kernel for the entire LTS lifecycle
- Cons: increased maintenance burden due to long support periods for all kernels

#### Hybrid Strategy 1: Native Kernel Full Support, Limited for Others

- The native kernel (K*n*) of V*n* is supported throughout the lifecycle of V*n*. Subsequent kernels receive a fixed support period (for example, 3 years).
- Logic: Guarantees the availability of the native kernel while providing newer kernels primarily for new hardware support.
- Pros: avoidance of continuous upgrades if the native kernel satisfies user demands
- Cons: periodic kernel upgrades for users requiring new hardware support

## Scope, Personnel, and Responsibilities

### Kernel SIG

- Necessary adjustments to the kernel repository
- Enhancements to kernel spec files
- Ongoing maintenance and support for subsequent kernel versions

### OS-Builder SIG

- Anaconda modifications
- oemaker modifications

### QA SIG

- Definition of testing strategies for multiple kernels
- Adaptation of kernel testing processes
- Adaptation of installation scenario testing processes

### Compatibility SIG

- Definition of compatibility strategies for multi-kernel support, including community compatibility testing and OSV certification to minimize impact For instance, the kernel-desktop package might not be subject to mandatory OSV certification compatibility requirements.
- Adaptation of compatibility testing processes

### Infra SIG

- Adaptation of OBS/EBS build environments
- Adaptation of image deliverables

### Doc SIG

- Updates to release notes and documentation

## Scope and Affected Software

The following software packages/repositories will be affected and require modification:

- Kernel
- Anaconda
- oemaker

## User Experience

Multi-kernel support significantly enhances user experience, allowing the use of newer hardware platforms while maintaining upper-layer application compatibility.

## Release Notes

Release notes should be updated to include a detailed explanation of this solution.

## Impacts on Upgrade and Compatibility

As the user-space API and ABI of the Linux kernel will continue to be stable ([source 1](https://www.kernel.org/doc/Documentation/process/stable-api-nonsense.rst), [source 2](https://en.wikipedia.org/wiki/Linux_kernel_interfaces)), introducing multiple kernels primarily affects hardware compatibility.

Multi-kernel support introduces variations in hardware compatibility for LTS versions. The following points require clarification:

- After a new kernel is released for an LTS version, should its hardware compatibility be retested, or can it inherit compatibility information from the previous version?
- Should compatibility information include the kernel version used during testing?
- On the compatibility web page, should the input and result displays differentiate between OS and kernel versions?
- Should OSV compatibility assessment require testing against newer kernels, or should it be optional?

## How to Test

To be determined.

## Implementation Plan

The plan is to implement and validate these changes in openEuler 24.0*X* (innovation version) and incorporate them into openEuler 24.03 LTS SP1.
