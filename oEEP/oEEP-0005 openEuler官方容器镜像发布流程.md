---
标题:     openEuler官方容器镜像发布流程
类别:     流程设计
摘要:     容器镜像发布流程定义与完善
作者:     姜逸坤 (yikunkero at gmail.com), 鲁卫军 (wjunlu217 at gmail.com)
状态:     活跃
编号:     oEEP-0005
创建日期:  2023-05-10
修订日期:  2023-10-23
---

## 动机/问题描述:

### 初衷
目前仅在openEuler官方网站发布了容器镜像的原始文件，如果未上传至任何三方docker镜像仓，开发者们不得不通过如下命令进行加载：
```
wget https://repo.openeuler.org/openEuler-20.03-LTS/docker_img/aarch64/openEuler-docker.aarch64.tar.xz
docker load < openEuler-docker.aarch64.tar.xz
docker run -ti openeuler-20.03-lts bash
```
这带来了[不太轻松的体验](https://gitee.com/openeuler/community/issues/I1K1BG)，同时，用户也无法基于此本地镜像与更多其他生态（例如Github Action、开源社区CI、容器应用镜像）进行集成。

2021年9月8日的[TC Meeting](https://gitee.com/openeuler/TC/blob/master/Meeting_Minutes/2021/Consolidated_Year_Meeting_Records.txt#L445-L447)(视频[0:22分起](https://www.bilibili.com/video/BV1ng411V7aB))汇报了《openEuler官方容器镜像发布》议题，同意创建[openeuler-docker-images](https://gitee.com/openeuler/openeuler-docker-images)仓库用于处理openEuler的Dockerfile和脚本。

自2021年发布至今(2023年5月)，[openeuler/openeuler](https://hub.docker.com/r/openeuler/openeuler)，已发布13个版本，累计下载5.1万次。

由此可见，发布在第三方容器镜像仓的openEuler官方容器镜像有真实的用户诉求。

### 用户案例
- 作为openEuler首次体验者，我希望能够快速体验openEuler操作系统，我只需要键入`docker run -ti openeuler/openeuler`便可以一键式体验openEuler。
- 作为openEuler的RPM软件包开发者，我可以通过基于容器镜像快速搭建rpm打包环境。
- 作为openEuler的上游社区，我希望能够基于openEuler容器镜像构建集成验证环境。例如OpenHPC社区使用openEuler容器镜像进行[CI验证](https://github.com/openhpc/ohpc/blob/d63e5573bd3f0986b51b5164de3ed514b778fa70/.github/workflows/validate.yml#L139)
- 作为openEuler基础设施的开发者，我们希望基于openEuler容器镜像构建基础设施所依赖的应用镜像。
- 作为openEuler的创新项目，我希望基于openEuler构建应用容器镜像，以便开发者能够快速体验到openEuler的创新项目。
- 作为openEuler的使用者，我希望能够在openEuler版本发布后，第一时间快速体验openEuler的新版本，同时，openEuler的更新我也能快速使用上。
- 作为OpenStack的上游开发者，我希望基于openEuler容器镜像定制OpenStack Kolla（容器组件）。
- 作为HPC领域的运维人员，部分软件依赖容器镜像（例如[Warewulf 4](https://warewulf.org/docs/development/quickstart/el7.html#pull-and-build-the-vnfs-container-and-kernel)），我希望通过容器镜像快速部署HPC集群。

### 相关SIG组及职责
- openEuler Release SIG：发布原始容器镜像的版本及更新版本，协助集成容器官方发布至Release流程中，随版本推送容器镜像，由每个版本的Release Manager负责发布至https://repo.openeuler.org/。
- openEuler Infra SIG：负责一键发布组件的设计、开发与维护，集成容器官方发布至Release流程中，由Infra SIG的维护者进行代码审核和最终镜像发布到容器镜像仓。
- openEuler Cloud Native SIG：负责原始容器镜像的裁剪、发布及代码审核。

### 本oEEP解决的问题
- 问题1：当前容器镜像发布至第三方仓流程未集成至Release流程，需要在版本发布后一天，人为触发。
- 问题2：当前Release发布的原始文件(openEuler-docker.aarch64.tar.xz)，仅在首个版本进行发布，而update版本未进行发布。
- 问题3：当前缺少AI容器发布到第三方仓库的流程，需要进行规范。

## 方案的详细描述:
### 1. 命名、标签规则
1. 名称: openeuler/openeuler
2. 标签: 以openEuler的版本名作为标签，例如：22.03-lts, 22.03-lts-sp1, 23.03
3. 特殊标签:
    - 20.03 → 代表20.03系列的推荐版本
    - latest → 代表最新的推荐版本，通常为LTS版本的长周期维护版本（例如22.03-lts-sp1）

| Version                 | Repo                                                                | Docker Tags   | Special Tags  |
|-------------------------|---------------------------------------------------------------------|---------------|---------------|
| openEuler-20.03-LTS     | https://repo.openeuler.org/openEuler-20.03-LTS/docker_img/          | 20.03-lts     |               |
| openEuler-20.03-LTS-SP1 | https://repo.openeuler.org/openEuler-20.03-LTS-SP1/docker_img/      | 20.03-lts-sp1 |               |
| openEuler-20.03-LTS-SP2 | https://repo.openeuler.org/openEuler-20.03-LTS-SP2/docker_img/      | 20.03-lts-sp2 |               |
| openEuler-20.03-LTS-SP3 | https://repo.openeuler.org/openEuler-20.03-LTS-SP2/docker_img/      | 20.03-lts-sp3 | 20.03         |
| openEuler-20.09         | https://archives.openeuler.openatom.cn/openEuler-20.09/docker_img/  | 20.09         |               |
| openEuler-21.03         | https://archives.openeuler.openatom.cn/openEuler-21.03/docker_img/  | 21.03         |               |
| openEuler-21.09         | https://archives.openeuler.openatom.cn/openEuler-21.09/docker_img/  | 21.03         |               |
| openEuler-22.03-LTS     | https://repo.openeuler.org/openEuler-22.03-LTS/docker_img/          | 22.03-lts     |               |
| openEuler-22.03-LTS-SP1 | https://repo.openeuler.org/openEuler-22.03-LTS-SP1/docker_img/      | 22.03-lts-sp1 | 22.03, latest |
| openEuler-22.03-LTS-SP2 | https://repo.openeuler.org/openEuler-22.03-LTS-SP2/docker_img/      | 22.03-lts-sp2 |               |
| openEuler-22.09         | https://repo.openeuler.org/openEuler-22.09/docker_img/              | 22.09         |               |
| openEuler-23.03         | https://repo.openeuler.org/openEuler-23.03/docker_img/              | 23.03         |               |

### 2. 代码仓库

https://gitee.com/openeuler/openeuler-docker-images

### 3. 代码合入与审核
1. 上传：上传dockerfile、meta.yml、脚本至openeuler-docker-images仓库。

特殊地，mete.yml用于制作AI容器镜像，其中每一对<key, value>的内容如下：
- tag (key): 镜像的标签
- Dockerfiel_path (value): 制作标签为tag镜像的Dockerfile保存路径

因此，AI容器镜像的标签不遵循上文对原始容器镜像的标签规范。

2. 审核：由Cloud Native SIG Maintainer进行审核后合入。

### 4. 发布流程
1. 由Release SIG发布"openEuler-docker.{arch}.tar.xz"至repo.openeuler.org。
- (已有发布) 对于首个版本发布，发布至：
https://repo.openeuler.org/openEuler-{VERSION}/docker_img/

- (已有发布) 对于update版本发布，发布至：
https://repo.openeuler.org/openEuler-{VERSION}/docker_img/update/YYYY-MM-DD/
；最新update版本覆盖拷贝至：https://repo.openeuler.org/openEuler-{VERSION}/docker_img/update/current/。

2. 通过"一键发布工具"获取发布的容器原始文件，发布至第三方容器仓库。
- "一键发布工具"——[EulerPublisher](https://gitee.com/openeuler/eulerpublisher)已经上线，由openEuler Infrastructure SIG维护，形式如下：
```
- 获取：
eulerpublisher container prepare --version ${VERSION}
- 推送：
eulerpublisher container push --version ${VERSION} --repo openeuler/openeuler
- 测试：
eulerpublisher container check --version ${VERSION} --repo openeuler/openeuler

- 一键获取、测试、推送
eulerpublisher container publish --version ${VERSION} --repo openeuler/openeuler
```
- 目前EulerPublisher通过Jenkins任务进行容器进行镜像发布(例如20.03、22.03维护版本的update[发布](https://jenkins.osinfra.cn/job/luweijun/job/eulerpublisher/47/))。

3. 通过"一键发布工具"，基于原始容器制作AI容器镜像，发布至第三方容器仓库。
```
# EulerPulisher一键发布AI容器镜像
eulerpublisher container publisher --meta meta.yml
```

