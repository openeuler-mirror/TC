---
Title:          Automatic DNF Package Download Optimization Based on User Location
Type:           Standard
Abstract:       Optimized mirror selection for enhanced download speeds using metalink configuration
Author:         zengchen1024 (chenzeng2@huawei.com)
Status:         Accepted
No:             oEEP-0004
Create_time:    2023-06-07
Update_time:    2023-06-07
---

## Background

With an increasing number of mirror sites serving openEuler, efficiently utilizing these sites to provide a seamless download experience for users has become crucial.
Currently, the **/etc/yum.repos.d/openEuler.repo** file only configures **baseurl** for each repository. Users are limited to downloading software from the site specified by **baseurl** unless they manually modify the configuration.
This can significantly impact download speeds, especially for users geographically distant from the designated site. Currently, 30 sites across Asia, Europe, and North America provide mirror services for openEuler.
Leveraging the **metalink** configuration in the DNF configuration file, we can enable users to download software from a geographically closer site, optimizing the download process.

## Objectives

Increasing the software package download speed for openEuler users.

## Solution Description

The solution involves implementing an openEuler metalink API and incorporating the **metalink** configuration into the DNF repository configuration. Additionally, we need to configure **metadata_expire** to ensure timely refresh of the local DNF cache.
In this context, "repository" refers to the configuration of a software repository within the **/etc/yum.repos.d/openEuler.repo** file. openEuler repositories include **OS**, **EPOL**, **debuginfo**, **everything**, **extras**, **source**, and **update**.
Below is an example configuration of a repository, showcasing the **metalink** and **metadata_expire** configurations:

```shell
[OS]
name=OS
baseurl=http://repo.openeuler.org/openEuler-22.03-LTS/OS/$basearch/
metalink=https://mirrors.openeuler.org/metalink?repo=$releasever/OS&arch=$basearch
metadata_expire=1h
enabled=1
gpgcheck=1
gpgkey=http://repo.openeuler.org/openEuler-22.03-LTS/OS/$basearch/RPM-GPG-KEY-openEuler
```

The URL associated with **metalink** returns XML data, as shown in the following example, which includes a list of available sites (the **resources** field) for the specific request. The entry of each site contains information such as supported protocols, geographic location, and priority. DNF utilizes this list to select the optimal site for downloading software packages.

```xml
<?xml version="1.0" encoding="utf-8"?>
<metalink version="3.0" xmlns="http://www.metalinker.org/" type="dynamic" pubdate="Wed, 07 Jun 2023 02:17:38 GMT" generator="mirrormanager" xmlns:mm0="https://mirror-manager.openeuler.org/">
 <files>
  <file name="repomd.xml">
   <mm0:timestamp>1685952673</mm0:timestamp>
   <size>3586</size>
   <verification>
    <hash type="md5">1e8a4244d3515a18d27ef65a75b66b6e</hash>
    <hash type="sha1">8cc28154ac6771dd92dbccecc4a79eea747c2cee</hash>
    <hash type="sha256">effd1df409e6936e88fd0312d7bd8efdcaccb1ca7bd9acf228b949e5a49c4620</hash>
    <hash type="sha512">187094f845f2e803108b81581aabc72b03cba95e32f3a0345a4554b5844171120d4e5726f3088d8a8950752e2de384def9573461475faa5b3e41ddc72adb6e16</hash>
   </verification>
   <mm0:alternates>
    <mm0:alternate>
      <mm0:timestamp>1685793651</mm0:timestamp>
      <size>3586</size>
      <verification>
       <hash type="md5">b44789ab70daf09337b16cf439608507</hash>
       <hash type="sha1">444c94d7de547ce5da34f85b19694204a3957676</hash>
       <hash type="sha256">7d08569f400022406ae86eddbe52239c29379caf8d2304d7a88832c93dc65ba4</hash>
       <hash type="sha512">c3cf2b55092779f6ad4596436aa883be8e527a3b1c13c545ce07581474f9f76f964809af4ae5d75e34018d5728fb79dd9d8a94fead6b701dbe46ce33d8055370</hash>
      </verification>
    </mm0:alternate>
   </mm0:alternates>
   <resources maxconnections="1">
    <url protocol="rsync" type="rsync" location="CN" preference="100">rsync://repo.openeuler.openatom.cn/openeuler/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="https" type="https" location="CN" preference="100">https://repo.openeuler.openatom.cn/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="https" type="https" location="CN" preference="99">https://mirrors.163.com/openeuler/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="https" type="https" location="CN" preference="98">https://mirrors.pku.edu.cn/openeuler/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="https" type="https" location="CN" preference="97">https://mirrors.nju.edu.cn/openeuler/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="rsync" type="rsync" location="CN" preference="97">rsync://root@mirrors.nju.edu.cn/openeuler/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="https" type="https" location="CN" preference="96">https://mirrors.tuna.tsinghua.edu.cn/openeuler/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="rsync" type="rsync" location="CN" preference="96">rsync://mirrors.tuna.tsinghua.edu.cn/openeuler/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="https" type="https" location="CN" preference="95">https://mirrors.aliyun.com/openeuler/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="https" type="https" location="CN" preference="94">https://mirror.lzu.edu.cn/openeuler/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="rsync" type="rsync" location="CN" preference="93">rsync://mirrors.hit.edu.cn/openeuler/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="https" type="https" location="CN" preference="93">https://mirrors.hit.edu.cn/openeuler/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="https" type="https" location="KR" preference="92">https://mirror.anigil.com/openeuler/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="http" type="http" location="KZ" preference="91">http://mirror.ps.kz/openeuler/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="rsync" type="rsync" location="HK" preference="90">rsync://119.8.63.103:873/openeuler/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
    <url protocol="https" type="https" location="HK" preference="90">https://repo.openeuler.org/openEuler-22.03-LTS/update/x86_64/repodata/repomd.xml</url>
   </resources>
  </file>
 </files>
</metalink>
```

The openEuler metalink API is implemented through the following two services.

### mirror manager

The [mirror manager website](https://mirror-manager.openeuler.org/) manages all mirror sites. Each site is required to register its metadata, including URL, geographic location, ASN, bandwidth, and maximum connections.
Two scheduled tasks collect information about each site:

- Task 1 retrieves directory information for all versions from the [main site](https://repo.openeuler.openatom.cn/), recording the update time and file information of each directory, and details of the **repomd.xml** file within the **repodata** directory.

- Task 2 uses the results from Task 1 as a reference, iterates through the remaining sites, checking if the **repodata** directory of each repository (such as **OS** and **EPOL**) for each version matches the main site.

### mirrorlist server

This service provides the metalink API. When a user requests a software package from a specific repository (such as **OS** and **EPOL**) and architecture (such as **x86_64** and **aarch64**), the metalink API follows this process:

1. Locates the **repodata** directory for the user's openEuler version, repository, and architecture.

2. Queries the database for site addresses that have a matching **repodata** directory with the main site.

3. Obtains the user's client IP address and uses geographic data, a country/region-continent mapping table, and a network range-ASN mapping table to determine the user's country/region, continent, and ASN.

4. Organizes the site addresses in the following order and generates XML data considering the site bandwidths:

```mermaid
graph LR;
A(Sites with the matching ASN) --> B(Sites in the same country/region);
B(Sites in the same country/region) --> C(Sites in the same continent);
```

## User Experience

Users will experience significantly faster software package download speeds.

## Release Notes

Modify **/etc/yum.repos.d/openEuler.repo** to add **metalink** and **metadata_expire** configurations for each repository.

## Impacts on Upgrade and Compatibility

Version upgrades must ensure that the **metalink** configuration matches the target version. If the directory structure of openEuler mirror sites remains consistent across versions, version upgrades will not affect the **metalink** configuration for the same repository.
If a repository is absent in a particular version, **/etc/yum.repos.d/openEuler.repo** will not have a corresponding configuration, and therefore, no **metalink** configuration.

## How to Test

Add the **metalink** configuration to each repository in the DNF configuration file and test using DNF or YUM commands, such as `yum makecache`.

## Dependencies

- The domain name of the metalink API should remain unchanged to avoid affecting the **metalink** configuration of older versions.

- The **metalink** configuration utilizes the **$releasever** DNF variable, which is derived from **src-openeuler/openEuler-release/generic-release.spec** using the following code. The rule for generating this variable should remain unchanged:

```shell
Provides: system-release = %{generic_version}%{generic_patch_level}_%{generic_patch_level_extend}
```

- The directory structure of openEuler mirror sites should remain consistent across versions to allow for the same **metalink** configuration to be used for the same repository across different versions. Currently, the directory structure of openEuler is consistent across versions.
For instance, all versions of openEuler have the following **repodata** directories:

```text
{openEuler version}/OS/aarch64/repodata
{openEuler version}/OS/x86_64/repodata
```

## Implementation Plan

We plan to support the **metalink** configuration in openEuler-22.03-LTS-SP2. Support for older versions will be discussed further.

## Contingency Plan

This feature enhancement only involves adding the **metalink** configuration to the DNF repository configuration. If this configuration fails, DNF will revert to using the **baseurl** configuration to download software packages, ensuring no degradation in user experience compared to the current state.

## Document Links

- [DNF Configuration Reference](https://dnf.readthedocs.io/en/latest/conf_ref.html)
- [openEuler Repository](https://docs.openeuler.org/en/docs/20.03_LTS/docs/Releasenotes/installing-the-os.html)
