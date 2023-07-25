---
标题:     openEuler智能补丁管理框架
类别:     流程设计
摘要:     现有缺陷修复流程新增支持热补丁制作、发布、巡检和修复
作者:     胡峰(solar.hu@huawei.com)
状态:     初始化
编号:     oEEP-0004 
创建日期: 2023-05-23
修订日期: 2023-07-11
---

## 动机/问题描述:
当前社区的CVE为冷补丁，补丁生效需要重启进程，kernel，glibc等底层组件需要重启系统，从而导致业务中断。
为了尽量减少对业务的影响，引入热补丁方案作为冷不丁的补充
同时完善openEuler安全漏洞和缺陷修复流程

## 收益分析
高分kernel安全缺陷可通过热补丁方式修复,避免业务中断

## 方案的详细描述：
1. 漏洞感知和QA提交缺陷报告
2. 对应sig组启动缺陷修复
3. release/security/maintainer确认是否要针对该缺陷发布热补丁
4. maintainer启动热补丁制作
5. release收集update发布清单，明确需要转测试的热补丁
6. QA完成补丁测试
7. release发布测试通过后的热补丁
8. 单机用户安装热补丁插件/集群用户安装apollo组件。配置热补丁源
9. 定期扫描或者手动执行缺陷扫描
10. 用户收到缺陷扫描报告
11. 用户选择需要修复的缺陷，系统激活补丁
12. 观察业务稳定性
13. 用户确认补丁状态（重启自动生效）

### 接口变更描述
- 修改cvrf.xml扩展热补丁信息
```
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
- 修改update.xml支持热补丁管理
```
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
			   //本次新增字段
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
- ci/cd支持热补丁制作,pr中新增热补丁制作接口：pr->issue->Hotpatch_matedata->release
   1. 开发人员在本次提交的pr评论区，执行makehotpatch来启动热补丁流程
   2. 后台会自动创建热补丁issue，用于跟踪整个热补丁过程，同时也作为后续update版本发布的依据
   3. 后台同时在Hotpatch仓库中以软件包名称（包括版本号），创建该热补丁仓库，并初始化热补丁管理xml
   4. 开发者通过提交补丁到该仓库中，完成热补丁修改和制作
   5. 最后修改issue内容和状态，便于update发布管理
- 新增hotpatch_meta仓库管理热补丁元数据
```
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

详细设计文档参考
https://gitee.com/openeuler/aops-apollo/blob/master/doc/design/aops-apollo%E7%89%B9%E6%80%A7%E8%AE%BE%E8%AE%A1%E6%96%87%E6%A1%A3(draft-for-hotpatch).md


## roadmap
|release time|Feature|openEuler release|
|-----|-----|------|
|2023-6-30|热补丁服务，热补丁巡检，热补丁生命周期管理|openEuler 22.03 LTS SP2|
|2023-9-30|热补丁支持可重复构建（时间一致性）|openEuler 22.09|
|2023-12-30|冷热补丁混合管理|openEuler 22.03 LTS SP3|
|2024-H1|热补丁可移植性|openEuler 2024 xxx|
