---
Title:          openEuler Software Quality Tiering and Execution Strategy
Type:           Process
Abstract:       Software quality tiering strategy
Author:         Fan Jiachen <fanjiachen3@huawei.com>
Status:         Initial
No:             oEEP-0017
Create_time:    2023-10-14
Update_time:    2023-10-14
---

## Background/Motivation/Objectives/Principles

As the openEuler community continues to grow, with an increasing number of SIGs and community software packages, various tiering strategies for community software have emerged. These include software compatibility tiering and software release tiering (BaseOS/Everything/EPOL/multiver).

To reduce the learning curve for developers regarding these standards and to unify the entry criteria, it is essential to solidify the quality of core software while streamlining requirements for some extended software packages. This document proposes a community-wide quality tiering strategy, jointly implemented by the QA and Package Management SIGs, that considers existing requirements and standards.

## Benefits for openEuler

* Clear quality tiers: Focusing on the image release scope (BaseOS/Everything/EPOL), this strategy applies detailed quality tiering only to the BaseOS scope. Everything and EPOL scopes will adhere to unified requirements and standards based on their release scope, simplifying the process for ecosystem developers.
* Unified standards and requirements: By clarifying maintenance, compatibility, and testing requirements based on the status of community quality expectations, this strategy provides community participants with easily accessible and documented requirements for each release version.

## Solution Description

### Software Package Tiering Strategy

|Tier|Characteristics|Example|
|:-|:-----|:-|
|L1|Critical software running on the data plane, components on the control and management planes that may cause network issues (such as upgrade-related software), and open source software in the attack path|openssh, dpdk|
|L2|Resident or frequently executed software on the control and management planes with active community involvement|systemd|
|L3|Mature community software running on the management plane with low usage frequency and stable functionality|bash, vim|
|L4|Software not used in production environments, such as tool software|make|

### Software Package Quality Tiering Strategy

|Tier|Maintenance Requirements|Release Repository|Compatibility Requirements|Testing Requirements|
|:-|:----|:--|:---|:----|
| L1 | 1. Routine <font color="#0000dd">weekly</font> synchronization of community patches/Fixes based on the <font color="#0000dd">SLA</font><br>2. Self-maintenance and evolution capabilities based on critical code flow analysis by maintainers or committers, with contribution back to the upstream community to establish upstream influence| <font color="#dd0000">BaseOS</font> | Package, API, and ABI compatibility maintained within an LTS version lifecycle, with <font color="#0000dd">no guarantee on cross-LTS version compatibility</font>| 1. Testing based on open source test suites<br>2. Supplementary special/DFX testing|
| L2 | 1. Routine bi-weekly synchronization of community patches<br>2. Fixes based on the <font color="#0000dd">SLA</font>| <font color="#dd0000">BaseOS</font> | Package, API, and ABI compatibility maintained within an SP version lifecycle, with <font color="#0000dd">no guarantee on cross-SP version compatibility</font>| 1. Testing based on open source test suites<br>2. Testing incorporating usage scenarios|
| L3 | 1. Routine monthly synchronization of community patches<br>2. Fixes based on the <font color="#0000dd">SLA</font>| <font color="#dd00000">BaseOS</font> | No compatibility guarantee| Testing based on open source test suites|
| L4 | Routine monthly synchronization of community fixes for high-risk vulnerabilities/critical bugs| <font color="#0000dd">Everything</font> | No compatibility guarantee| Testing based on open source test suites|
| N/A| Passive response to community vulnerabilities/bugs| EPOL | No compatibility guarantee| RPM package-level functionality guarantee|

## Implementation Plan

### Activity - Package Introduction

| Attribute| Description|
|:------|:---|
| Name| Package introduction|
| Description| Create a repository for the package, create branches, and submit the source code to the corresponding src-openEuler repository.|
| Instructions| 1. Developers submit a package introduction pull request (PR) to openEuler/community (refer to Input 1 for PR content).<br>2. The Technical Committee (TC) and maintainers of the package owner SIG review and merge the PR (a src-openEuler repository is automatically created after the PR is merged).<br>3. Developers submit code to the corresponding src-openEuler repository.<br>4. Maintainers of the package owner SIG review and merge the PR.<br>5. The build system automatically retrieves the package changes and applies them to the Factory build process.|
| Responsible role| Package owner SIG (approves repository creation; takes over package maintenance)|
| Participating roles| Package management SIG (defines repository creation and RPM source code encoding specifications)<br>TC (approves and merges repository creation PR)<br>Developers (lead repository creation and code initialization)|
| Input| 1. Package configuration table (package name, upstream information, branch information, SIG information)<br>2. Package source code (spec, source code, patch, and other files)|
| Output| N/A |

### Activity - Package Entry into Version Software Pool

| Attribute| Description|
|:------|:---|
| Name| Package entry into version software pool (that is, Factory -> Epol/Mainline)|
| Description| After a package repository is created and the build in the Factory project is stable and successful, the package is moved to the formal version software project (Mainline, EPOL, etc.).|
| Instructions| 1. The package management SIG regularly (quarterly recommended) reviews the package build status within the Factory project and compiles a list of packages eligible for the version software pool, notifying the package owner SIG.<br>2. Based on this list, the package owner SIG decides whether to include the package in the version and determines its quality tier. The package is then added to the corresponding branch based on its quality tier, and a PR is submitted to the openEuler/release-management repository.<br>3. The Release Management (RM) SIG maintainer reviews and merges the PR (packages included in the ISO scope require a designated maintenance committer).<br>4. The build system automatically applies the PR changes in the project, completing build project modification for the package. |
| Responsible role | Package owner SIG (leads the migration operation) |
| Participating roles | Package management SIG (inspects packages meeting the criteria and notifies the corresponding SIGs to migrate)<br>RM SIG (approves and merges package project modification PRs)<br> CI/CD SIG (defines configuration file specifications, implements configuration, and builds system project capabilities) |
| Input | 1. Package version information (quality tier, build project, maintenance committer)<br>2. Baseline for version software scope |
| Output | N/A |

### Activity - Version Software Baseline

| Attribute| Content|
|:------|:---|
| Name | Version software baseline (that is, software scope baseline for the innovation, initial LTS, and LTS-SP versions) |
| Description | Before the full build of the branch during the community version release planning process, clarify the software scope for the version, including the baseline for the version launch phase and software removal during the version release phase. |
| Instructions | 1. Based on the build status of packages within the version software pool (Mainline and EPOL projects for the innovation/initial LTS version, LTS-Next related projects for the LTS-SP version), the Package management SIG compiles a list of packages eligible for the version software pool and submits PRs to create version branches in openEuler/community and projects in openEuler/release-management.<br>2. The TC reviews and merges the branch creation PR, while the RM SIG reviews and merges the project creation PR.<br>3. The community CI automatically creates a PR based on the branch creation to create a branch in src-openEuler. The build system automatically creates a PR based on the project creation to generate a build project in the build system.<br>4. The Package Management SIG continuously tracks the status of software packages in the version and proposes changes to the version software scope during the release phase. |
| Responsible role | Package Management SIG (leads and defines the version software package baseline) |
| Participating roles | RM SIG (approves and merges project creation PRs)<br> TC SIG (approves and merges branch creation PRs)<br> CI/CD SIG (defines configuration file specifications, implements configuration, and builds system project capabilities) |
| Input | 1. Baseline for version software scope |
| Output | N/A |

### Activity - Version Software Selection and Upgrade

| Attribute| Content|
|:------|:---|
| Name | Version software selection and upgrade |
| Description | During the period between version release planning, the software within the version software pool undergoes selection, upgrade, and retirement. |
| Instructions | 1. The Package Management SIG regularly (quarterly recommended) reviews the upstream information of packages based on the version branch scope and notifies the package owner SIG of package upgrade/retirement suggestions through issues.<br>2. Based on these suggestions, the package owner SIG decides whether to upgrade/retire the package. In the package is to be upgraded, the SIG submits an upgrade PR and completes the upgrade difference analysis, submitting a PR for the analysis as well. If the package is to be retired, the SIG completes the dependency truncation/replacement within the version, announces it on the community mailing list, and transfers the package ownership to the Recycle SIG.<br>3. The upgrade PR is reviewed and merged by the package owner SIG.<br>4. The PR for transferring the package ownership to the Recycle SIG is reviewed and merged by the TC. If software retirement affects the version build, the Package Management SIG has the right to organize discussions with the package owner SIG and extend the maintenance period within a certain limit. |
| Responsible role | Package owner SIG (develops package upgrades based on selection suggestions) |
| Participating roles | Package Management SIG (provides selection and retirement suggestions based on software upstream information)<br>TC (approves and merges PRs for transferring packages to the Recycle SIG) |
| Input | Package selection suggestions (upgrade or retire) |
| Output | N/A |

### Activity - Maintenance Version Quality Tracking & Quality Tier Change Suggestions

| Attribute| Content|
|:------|:---|
| Name | Maintained version quality tracking and quality tier change suggestions |
| Description | During the continuous maintenance phase of LTS and LTS-SP versions, this activity continuously monitors whether the software package meets the quality requirements based on its quality tier. |
| Instructions | 1. Based on the version software baseline, the Package Management SIG reviews core metrics such as CVE fixes, bug fixes, and issue resolution. Packages that do not meet the quality requirements are statistically analyzed and announced to the community.<br>2. The Package Management SIG organizes discussions with RM, QA, and the package owner SIGs to determine the handling policy for packages that do not meet the quality tier (for example, maintenance responsibility transfer, quality downgrade, project modification, or software retirement). Packages that show no improvement for three consecutive months will be downgraded. Packages with no improvement for six consecutive months will be announced to the community and retired after one month. |
| Responsible role | Package Management SIG (leads and reviews the quality compliance of version software) |
| Participating roles | RM SIG (approves and merges PRs for changing release scope)<br> TC (approves and merges PRs for retiring packages) |
| Input | Maintained version software scope and quality compliance status (list of non-compliant software and rectification suggestions) |
| Output | N/A |
