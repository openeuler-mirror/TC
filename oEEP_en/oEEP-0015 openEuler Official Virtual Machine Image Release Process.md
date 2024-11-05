---
Title:          openEuler Official Virtual Machine Image Release Process
Type:           Process
Abstract:       Release process for openEuler official virtual machine images
Author:         Lu Weijun <wjunlu217@gmail.com>
Status:         Active
No:             oEEP-0015
Create_time:    2023-10-24
Update_time:    2023-10-24
---

## Motivation/Problem

Currently, virtual machine (VM) images published on the openEuler website face the following issues:

1. Missing basic software: Images lack pre-installed software like `wget`, `hostname`, and `unzip`, leading to frequent failures when users deploy applications on openEuler for the first time.
2. Lack of cloud adaptation: Images lack crucial `cloud-init` components, causing frequent import failures into public cloud platforms.
3. No update release process: Users cannot experience the latest features of each openEuler version without an update mechanism.

This proposal aims to standardize the process for determining pre-installed software in VM images, enabling users to customize images based on their needs, and releasing updated versions.

### Issues Addressed by this oEEP

- How pre-installed software is determined for each VM image release and how users can customize the published image for their needs
- Process for releasing updated versions of the VM images, addressing the current limitation of only releasing the initial version

## Solution Description

### 1. Related SIG and Responsibilities

1. Gatekeeper SIG: Responsible for building, tailoring official VM images, and merging code for building components. Code repositories:
    - <https://gitee.com/openeuler/openeuler-os-build>
    - <https://gitee.com/openeuler/CreateImage/>
2. Release SIG: Responsible for releasing the initial and updated versions of VM images. The release manager for each version reviews and approves changes to the pre-installed software scope and publishes images to <https://repo.openeuler.org/>.
3. Infrastructure SIG: Responsible for designing, developing, and maintaining the one-click publishing component, EulerPublisher. Based on the images published at <https://repo.openeuler.org/>, it provides users with the ability to customize the pre-installed software in images for different usage scenarios.

### 2. Pre-installed Software Changes

1. The RPM package lists containing pre-installed software packages within VM images are stored in the <https://gitee.com/openeuler/release-management> repository in the following format:

    ```text
    #RPM package list example
    kexec-tools
    net-tools
    iproute
    ...
    ```

2. To modify the pre-installed software scope, the RPM package list must be changed through a pull request. The release manager reviews and approves the changes before merging for it to take effect.

### 3. Release Process

The Release SIG releases **openEuler-**_{VERSION}_**-**_{ARCH}_**.qcow2.xz** to <https://repo.openeuler.org> as follows:

- **(Existing releases)**: Initial versions are released to <https://repo.openeuler.org/openEuler-{VERSION}/virtual_machine_img/>.

- **(New releases)**: Updated versions are released to <https://repo.openeuler.org/openEuler-{VERSION}/virtual_machine_img/update/YYYY-MM-DD/>.
The latest update version is also copied to <https://repo.openeuler.org/openEuler-{VERSION}/virtual_machine_img/update/current/>.
