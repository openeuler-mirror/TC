---
Title:          Non-Upstream Multi-Architectural Code Submission Rules for openEuler Software Packages
Type:           Process
Abstract:       Non-upstream support for software packages in openEuler repositories
Author:         Ye Qinglong <yeqinglong@kylinsec.com.cn>, Hu Xinwei <shinwell_hu@openeuler.sh>
Status:         Initial
No:             oEEP-0012
Create_time:    2023-08-17
Update_time:    2023-08-30
---

## Motivation/Problem

### Background

openEuler supports numerous architectures. However, some software components lack complete upstream support for certain architectures. In other cases, while upstream support exists, the specific software version used in a particular openEuler release might not fully support those architectures.
Our objectives are:

- To enable openEuler software to support different architectures, regardless of upstream support status
- To maintain a single codebase for openEuler software of different architectures (ensuring identical code after the **%prep** step of rpmbuild) to facilitate consistent bug fixing and vulnerability fixing across all architectures
- To ensure that the package selection and evolution of openEuler are not hindered by the absence of upstream acceptance for architecture enablement code
- To allow openEuler packages to seamlessly incorporate upstream updates for architecture enablement code through upgrades, eliminating the need for separate maintenance

To provide comprehensive and sustainable architecture support for software packages in openEuler repositories, the community needs to establish corresponding specifications and process requirements.

### Current Issues

Currently, architecture-related patches lack standardized constraints within the community. The acceptance criteria for such patches vary across different package management practices, leading to issues such as:

- Repetitive modifications for different architectures in the same location, creating unnecessary dependency complexities and hindering maintainability
- Architecture-specific patching, resulting in code discrepancies across different architectures and potential risks of inconsistency between code and binary files
- Lack of clear distinction between upstream backports and patches created during development, making it difficult to analyze package evolution
- Lack of clear differentiation between architecture-related patches and other patches, leading to inconsistent application order across different packages

### Related SIGs and Responsibilities

- All SIGs: Requirements related to architecture patch incorporation should be added to the pull request (PR) review checklist of SIGs and implemented during PR review.
- Infrastructure SIG: enhance continuous integration (CI) to improve sensitivity to conditional build macros like **ifarch** during the build process.

## Solution Description

### Code Requirements

1. In principle, patches should not target specific architectures. If the **ifarch** macro appears in an RPM spec file, a preceding comment must clearly explain the reason and current technical limitations.
2. Patch names should explicitly state whether they have been accepted by the upstream community or backported from a specific version to facilitate processing during version upgrades. For example:

    ```text
    Patch2:  backport-fiels-version-info.patch
    ```

3. In principle, new architecture-related code files and patches modifying existing architecture-related code should be separated. Avoid combining code modification and file additions within a single patch. For example:

    ```text
    Patch2:  0002-add-riscv-support-not-upstream-modified-files.patch
    Patch3:  0003-add-riscv-support-not-upstream-new-files.patch
    ```

4. Ideally, patches modifying architecture-related code should focus on a limited number of files. Split large patches using tools like split-patch, with a maximum recommended limit of 5 file modifications per patch (subject to adjustment). For example:

    ```text
    Patch1001: 1001-add-riscv-support-not-upstream-modified-files-1.patch
    Patch1002:  1002-add-riscv-support-not-upstream-modified-files-2.patch
    ```

5. When applying patches for different architectures to the same file, prioritize patches for more stable architectures. Package maintainers can make adjustments based on specific circumstances but should ensure consistency in patch application order across different files.

### Patch Application Order

**Note:** The numerical order (_XXXX_) in patch names within an RPM spec file (for example, **Patch**_XXXX_) does not dictate the application order. The following example demonstrates an incorrect order:

```text
Patch0002: patch-2.diff
Patch0001: patch-1.diff
```

In this case, **patch-2.diff** will be applied before **patch-1.diff**. Therefore, openEuler mandates that the declared order of patches must align with their numerical sequence. The correct declaration order for the example above should be:

```text
Patch0001: patch-1.diff
Patch0002: patch-2.diff
```

### Patch Declaration Order

1. Patches from the same upstream version: 0001-0999
2. Architecture-related patches: 1000-1999
3. Reserved patch numbers: 2000-2999
4. CVE patches and backports from other upstream versions: 3000-4999
5. openEuler-specific feature patches: 5000 and above

### Package CVE Patches

CVE patches are declared after architecture patches. openEuler vulnerability fixes are inherently required to consider support for all architectures. CVE patches should be adapted and refreshed to support all openEuler-supported architectures. This approach allows for easy detection of conflicts when both architecture and CVE patches modify the same file. However, there is a possibility of a CVE patch addressing independent files for other architectures. In such cases, the maintainer should consult with the architecture patch submitter to confirm the patch. If fixing a specific architecture proves excessively difficult and hinders the release of patches for other architectures, temporary workarounds using **ifarch** during the build process are acceptable. However, persistent **ifarch** usage should be treated as a CI issue, prompting the maintainer and committer to address it. Architectures with persistent issues might not be suitable for official support in openEuler LTS versions.

### Package Upgrades

All upstream patches are prioritized over architecture-specific declarations, ensuring that upstream development takes precedence over unique architecture support of openEuler. If an upgrade directly provides support for relevant architectures, the corresponding patches can be dropped after the upgrade. Otherwise, architecture patches should be refreshed in conjunction with upstream upgrades. The submitters of architecture patches are responsible for refreshing them during upgrades. If a specific architecture cannot keep pace with upstream evolution, it might not be suitable for official support in openEuler LTS versions.

### Packaging Specifications

<https://gitee.com/openeuler/community/blob/master/en/contributors/packaging.md>
