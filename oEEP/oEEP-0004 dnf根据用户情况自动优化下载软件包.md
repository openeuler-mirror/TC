---
标题:     dnf根据用户情况自动优化下载软件包
类别:     特性变更
摘要:     通过配置metalink为用户选择更优的镜像站点，从而提高下载速度
作者:     陈曾 (zengchen1024 at chenzeng2@huawei.com)
状态:     接纳
编号:     oEEP-0004
创建日期: 2023-06-07
修订日期: 2023-06-07
---

## 背景

随着越来越多的镜像站点为openEuler提供镜像服务，如何将这些站点利用起来为用户提供良好的下载体验成为一个迫切的问题。
当前，/etc/yum.repos.d/openEuler.repo 文件中每个Repo只配置了baseurl，用户如果不手动修改配置则只能去baseurl指向
的站点下载软件。如果用户离这个站点的地理位置较远，其下载软件包的速度将会非常受影响。实际上，当前有30个站点为openEuler
提供了镜像服务，这些站点分布在亚洲、欧洲和北美。用户的身边可能就有一个站点，从这个较近的站点下载软件就可以避免舍近求远。
dnf配置文件中，metalink配置项的作用就是为此而设计。

## 目标

提高openEuler用户下载软件包的速度。

## 方案描述

实现openEuler的metalink API，并在dnf的Repo配置中增加**metalink**配置项；另外需要配置**metadata_expire**，确保dnf能在合理时间内刷新本地cache。
这里的Repo是指在/etc/yum.repos.d/openEuler.repo文件中一个软件仓的配置。openEuler的Repo包括OS，EPOL，debuginfo，everything，extras，source， update等。
一个Repo的配置示例如下，示例中metalink和metadata_expire即是本次需要增加的配置项。
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
metalink对应的URL会返回一个xml格式的数据，其内容示例如下。从示例中可知，该文件包含了针对本次请求的可用的站点列表（resources字段）。每个站点包含了支持的协议，所在的国家，优先级等。这些站点列表可以被dnf使用，从中选择站点下载软件包。
```shell
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

通过如下两个服务实现openEuler的metalink API。

### mirror manager

[mirror manager 网站](https://mirror-manager.openeuler.org/)管理所有的站点，每个站点需要注册其元数据信息，包括站点的URL，所在国家，ASN，带宽，最大连接数等。通过两个
定时任务统计各个站点的信息。

- 任务一，从[**主站点**](https://repo.openeuler.openatom.cn/)拉取所有版本的目录信息，记录各个目录的更新时间和目录下的文件信息，以及repodata目录下repomd.xml文件的信息；

- 任务二，以任务一的结果作为参照物，遍历其余各个站点，记录各个版本的各个Repo（OS、EPOL等）下的repodata目录是否与主站一致。

### mirrorlist server

该服务提供了metalink API。当用户需要查询某个Repo（OS、EPOL等）下的某个架构（x86_64、aarch64等）的软件包时，metalink API的处理流程如下。

1. 找到用户的openEuler版本在这个Repo和架构下的repodata目录。

2. 查询数据库获取各个站点地址，这些站点的这个repodata目录与主站点是保持一致的。

3. 获取用户的客户端IP地址，并根据地理信息数据，国家-大洲映射表，以及网段-ASN映射表查询该用户所在的国家，大洲和ASN。

4. 按照如下顺序来组织各个站点地址，并参考站点的带宽信息生成xml格式的数据。
```mermaid
graph LR;
A(与客户端的ASN一致的站点) --> B(与客户端同一个国家的站点);
B(与客户端同一个国家的站点) --> C(与客户端同一个大洲的站点);
```

## 用户体验

用户下载软件包的速度将得到较大提升。

## 发布说明

修改/etc/yum.repos.d/openEuler.repo，对每个Repo增加metalink和metadata_expire配置。

## 升级与兼容性影响

版本升级需要确保metalink的配置是与目标版本匹配的。如果能保证openEuler镜像站点的目录结构在各个版本之间保持不变，那么版本升级不会影响同一个Repo的metalink配置。
如果某个版本缺少某个Repo，则/etc/yum.repos.d/openEuler.repo没有对应的Repo配置，进而也不会有对应的metalink配置。

## 如何测试

在dnf的配置文件中，对每个Repo增加metalink配置，使用dnf或yum命令进行测试，比如yum makecache。

## 依赖关系

- metalink API的域名应保持不变，否则会影响老版本的metalink配置。

- metalink 配置中使用了dnf变量$releasever。这个变量的值来自于src-openeuler/openEuler-release/generic-release.spec，代码如下，该变量的生成规则应保持不变。
```shell
Provides: system-release = %{generic_version}%{generic_patch_level}_%{generic_patch_level_extend}
```

- openEuler镜像站点的目录结构应在各个版本之间保持不变，这样才能在不同的版本之间对同一个Repo使用相同的metalink配置；否则每个版本只能使用各自的metalink配置。当前openEuler各个版本的目录结构是一致的，
比如所有版本的OS都有如下的repodata目录。
```shell
{openEuler version}/OS/aarch64/repodata
{openEuler version}/OS/x86_64/repodata
```

## 实施计划

计划在openEuler-22.03-LTS-SP2版本中支持metalink的配置。对老版本支持metalink需要进一步讨论。

## 应急计划

此次特性变更只是在dnf的Repo配置中增加metalink配置，如果此配置项失效，dnf仍然会使用baseurl的配置来下载软件包，用户的体验不会比当前要差。

## 文档链接

- [dnf 配置参考](https://dnf.readthedocs.io/en/latest/conf_ref.html)
- [openEuler Repo](https://docs.openeuler.org/zh/docs/20.03_LTS/docs/Releasenotes/%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85.html)
