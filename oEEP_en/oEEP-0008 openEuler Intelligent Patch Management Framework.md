---
Title:          openEuler Intelligent Patch Management Framework
Type:           Process
Abstract:       Enhancement of existing vulnerability fixing process for hot patch management
Author:         Hu Feng <solar.hu@huawei.com>
Status:         Initial
No:             oEEP-0008
Create_time:    2023-05-23
Update_time:    2023-07-11
---

## Motivation/Problem

Currently, Common Vulnerabilities and Exposures (CVEs) in the openEuler community are addressed with cold patches that require process restarts to take effect. This necessitates system reboots for core components like the kernel and glibc, leading to service interruptions.
To minimize service downtime, we propose introducing a hot patching mechanism to complement the existing cold patching mechanism.
This will also enhance openEuler's overall security vulnerability fixing process.

## Benefits

Critical kernel security vulnerabilities can be addressed with hot patches, preventing service interruptions.

## Solution Description

1. A vulnerability is identified and reported to the relevant SIGs through a QA process.
2. The SIGs initiate the vulnerability fixing process.
3. The Release Management SIG, Security Committee, and related SIG maintainers assess the vulnerability and determine if a hot patch is necessary.
4. Upon approval, the related SIG maintainers create the hot patch.
5. The Release Management SIG compiles an update list, specifying hot patches for testing.
6. The QA SIG tests the patches.
7. The Release Management SIG release tested and approved hot patches.
8. User-end patch deployment:
    - Individual users install a hot patch plugin.
    - Cluster users install the Apollo component and configure hot patch sources.
9. Periodic or manual vulnerability scans are conducted.
10. Users receive defect scan reports.
11. Users choose which vulnerabilities to fix. The system activates the selected patches.
12. Uses monitor the service system stability.
13. Users confirm the patch status (reboots automatically apply the patches).

### Interface Changes

- **cvrf.xml** enhancement for hot patch information

```xml
<cvrfdoc xmlns="http://www.icasi.org/CVRF/schema/cvrf/1.1" xmlns:cvrf="http://www.icasi.org/CVRF/schema/cvrf/1.1">
    <DocumentTitle xml:lang="en">An update for patchelf is now available for openEuler-22.03-LTS and openEuler-22.03-LTS-SP1</DocumentTitle>
    <DocumentType>Security Advisory</DocumentType>
    <DocumentPublisher Type="Vendor">
        ...
    </DocumentPublisher>
    <DocumentTracking>
        ...
    </DocumentTracking>
    <DocumentNotes>
        ...
    </DocumentNotes>
    <DocumentReferences>
        ...
    </DocumentReferences>
    <ProductTree xmlns="http://www.icasi.org/CVRF/schema/prod/1.1">
        <Branch Type="Product Name" Name="openEuler">
            <FullProductName ProductID="openEuler-22.03-LTS" CPE="cpe:/a:openEuler:openEuler:22.03-LTS">openEuler-22.03-LTS</FullProductName>
        </Branch>
        <Branch Type="Package Arch" Name="aarch64">
            <FullProductName ProductID="patchelf-0.16.0-1" CPE="cpe:/a:openEuler:openEuler:22.03-LTS">patchelf-0.16.0-1.oe2203.aarch64.rpm</FullProductName>
        <Branch Type="Package Arch" Name="src">
            <FullProductName ProductID="patchelf-0.16.0-1" CPE="cpe:/a:openEuler:openEuler:22.03-LTS">patchelf-0.16.0-1.oe2203.src.rpm</FullProductName>
        </Branch>
        <Branch Type="Package Arch" Name="x86_64">
            <FullProductName ProductID="patchelf-debuginfo-0.16.0-1" CPE="cpe:/a:openEuler:openEuler:22.03-LTS">patchelf-debuginfo-0.16.0-1.oe2203.x86_64.rpm</FullProductName>
        </Branch>
    </ProductTree>
    <HotPatchTree xmlns="http://www.icasi.org/CVRF/schema/prod/1.1">
        <Branch Type="Product Name" Name="openEuler">
            <FullProductName ProductID="openEuler-22.03-LTS" CPE="cpe:/a:openEuler:openEuler:22.03-LTS">openEuler-22.03-LTS</FullProductName>
        </Branch>
        <Branch Type="Package Arch" Name="aarch64">
            <FullProductName ProductID="patchelf-0.16.0-1" CPE="cpe:/a:openEuler:openEuler:22.03-LTS">patch-patchelf-0.16.0-1.oe22030-HP001.aarch64.rpm</FullProductName>
        <Branch Type="Package Arch" Name="src">
            <FullProductName ProductID="patchelf-0.16.0-1" CPE="cpe:/a:openEuler:openEuler:22.03-LTS">patch-patchelf-0.16.0-1.oe22030-HP001.src.rpm</FullProductName>
        </Branch>
        <Branch Type="Package Arch" Name="x86_64">
            <FullProductName ProductID="patchelf-debuginfo-0.16.0-1" CPE="cpe:/a:openEuler:openEuler:22.03-LTS">patch-patchelf-0.16.0-1.oe22030-HP001.x86_64.rpm</FullProductName>
        </Branch>
    </HotPatchTree>
    <Vulnerability xmlns="http://www.icasi.org/CVRF/schema/vuln/1.1" Ordinal="1">
        ...
    </Vulnerability>
</cvrfdoc>
```

- **update.xml** modification for hot patch management

```xml
<?xml version="1.0" encoding="UTF-8"?>
<updates>
    <update from="openeuler.org" type="security/bugfix/feature" status="stable">
        <id>openEuler-SA-2021-1502</id>
        <title>An update for polkit is now available for openEuler-20.03-LTS-SP3</title>
        <severity>Important</severity>
        <release>openEuler</release>
        <issued date="2022-01-27"></issued>
        <references>
            <reference href="https://nvd.nist.gov/vuln/detail/CVE-2021-4034" id="CVE-2021-4034" title="CVE-2021-4034" type="cve"></reference>
        </references>
        <description>xxxxxxxxxxxxx</description>
        <pkglist>
            <collection>
                <name>openEuler</name>
                <package arch="aarch64/noarch/x86_64" name="polkit" release="9.oe1" version="0.116">
                    <filename>polkit-0.116-9.oe1.aarch64.rpm</filename>
                </package>
            </collection>
            // New field
            <hot_patch_collection>
                <name>openEuler</name>
                <package arch="aarch64" name="polkit" release="9.oe1" version="0.116">
                    <filename>polkit-0.116-9.oe1.aarch64.rpm</filename>
                </package>
                <package arch="noarch" name="polkit-help" release="9.oe1" version="0.116">
                    <filename>polkit-help-0.116-9.oe1.noarch.rpm</filename>
                </package>
            </hot_patch_collection>
        </pkglist>
    </update>
</updates>
```

- CI/CD integration for hot patch creation: A new interface for hot patch creation is added to the pull request (PR) workflow (PR -> Issue -> hotpatch_meta -> release).
   1. Developers initiate the hot patch process by submitting the `makehotpatch` command in the PR comment section.
   2. A hot patch issue is automatically created for tracking and serves as a reference for future update releases.
   3. A repository for the hot patch, named after the software package (including the version number), is created in the hotpatch_meta repository, and an XML file for hot patch management is initialized.
   4. Developers commit patches to this repository, completing the hot patch modification and creation process.
   5. The issue content and status are updated to facilitate update release management.
- hotpatch_meta repository for metadata management: A new repository, hotpatch_meta, is introduced to manage hot patch metadata.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<hotpatchdoc xmlns="https://gitee.com/openeuler/hotpatch_data" xmlns:cvrf="https://gitee.com/openeuler/hotpatch_data">
    <DocumentTitle xml:lang="en">Managing Hot Patch Metadata</DocumentTitle>
    <HotPatchList>
    <Package Name="kernel-xxxxxxx" Type="cve/bug" statue="i/f/s">
        <hotpatch version="1">
            <SRC_RPM>download_link</SRC_RPM>
            <Debug_RPM>download_link</Debug_RPM>
            <Patch>xxxxxx.patch</Patch>
        </hotpatch>
        <hotpatch version="2">
            <SRC_RPM>download_link</SRC_RPM>
            <Debug_RPM>download_link</Debug_RPM>
            <Patch>xxxxxx.patch</Patch>
        </hotpatch>
    </Package>
    </HotPatchList>
</hotpatchdoc>
```

## Roadmap

| Release Date       | Feature                                                 | openEuler Version |
| ------------------ | ------------------------------------------------------- | ----------------- |
| June 30, 2023      | Hot patch service, inspection, and lifecycle management | 22.03 LTS SP2     |
| September 30, 2023 | Repeatable hot patch builds (time consistency)          | 22.09             |
| December 30, 2023  | Hybrid management of cold and hot patches               | 22.03 LTS SP3     |
| 2024 H1            | Hot patch portability                                   | 24.xx             |
