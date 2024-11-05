---
Title:          openEuler Official Container Image Release Process
Type:           Process
Abstract:       Container image release process definition and improvement
Author:         Jiang Yikun <yikunkero@gmail.com>, Lu Weijun <wjunlu217@gmail.com>
Status:         Active
No:             oEEP-0005
Create_time:    2023-05-10
Update_time:    2023-10-23
---

## Motivation/Problem

### Background

Currently, openEuler only publishes raw container image files on the official website. These files are not uploaded to any third-party Docker image repositories. Developers have to load them using the following commands:

```shell
wget https://repo.openeuler.org/openEuler-20.03-LTS/docker_img/aarch64/openEuler-docker.aarch64.tar.xz
docker load < openEuler-docker.aarch64.tar.xz
docker run -ti openeuler-20.03-lts bash
```

This results in a [cumbersome user experience](https://gitee.com/openeuler/community/issues/I1K1BG) and prevents users from integrating these local images with other ecosystems like Github Actions, open source CI tools, and container applications.

On September 8, 2021, the openEuler Technical Committee (TC) meeting ([minutes](https://gitee.com/openeuler/TC/blob/master/Meeting_Minutes/2021/Consolidated_Year_Meeting_Records.txt#L445-L447) discussed the "openEuler official container image release" proposal. The meeting agreed to create the [openeuler-docker-images](https://gitee.com/openeuler/openeuler-docker-images) repository to manage openEuler's Dockerfiles and related scripts.

Since its launch in 2021, [openeuler/openeuler](https://hub.docker.com/r/openeuler/openeuler) has released 13 versions (as of May 2023) with over 51,000 downloads, demonstrating a clear user demand for official openEuler container images published on third-party repositories.

### User Cases

- First-time openEuler user can quickly experience the OS by simply running `docker run -ti openeuler/openeuler`.
- openEuler RPM package developers can leverage container images to rapidly set up RPM packaging environments.
- Users of the upstream community of openEuler can use openEuler container images to build and validate integration. For example, the OpenHPC community utilizes these images for [CI validation](https://github.com/openhpc/ohpc/blob/d63e5573bd3f0986b51b5164de3ed514b778fa70/.github/workflows/validate.yml#L139).
- openEuler infrastructure developers rely on openEuler container images to build application images for community infrastructure.
- openEuler innovation projects can be built as container images based on openEuler to allow developers to quickly experience them.
- openEuler users can have immediate access to new openEuler versions upon release and quick access to updates.
- OpenStack upstream developers can use openEuler container images to customize OpenStack Kolla (container components).
- HPC O&M engineers rely on container images for certain software (such as [Warewulf 4](https://warewulf.org/docs/development/quickstart/el7.html#pull-and-build-the-vnfs-container-and-kernel)) and require them for rapid HPC cluster deployment.

### Related SIGs and Responsibilities

- Release SIG: releases new and updated versions of the raw container images. This SIG assists in integrating the official container images into the overall release workflow and is responsible for pushing container images with each release. The release manager for each version publishes the images to <https://repo.openeuler.org/>.
- Infrastructure SIG: designs, develops, and maintains the one-click publishing tool. This SIG integrates the official container images into the overall release workflow. Infrastructure SIG maintainers conduct code reviews and publish the final images to container image repositories.
- Cloud Native SIG: tailors, releases, and reviews the code for raw container images.

### Problems Addressed by This oEEP

- The current process of publishing official container images to third-party repositories is not integrated into the release workflow. The process can only be manually triggered a day after the version release.
- Currently, only the initial release of the raw container image file (**openEuler-docker.aarch64.tar.xz**) is published. Update releases are not published.
- Currently, there is no defined standard for publishing application-specific container images to third-party repositories.

## Solution Description

### 1. Naming and Tagging Conventions

This oEEP defines two types of container images:

- **Base container images** are the official container image published to **<https://repo.openeuler.org/openEuler-{VERSION}/docker_img/>**. These images contain a minimal set of base software.
- **Application container images** are built on base images with additional software installed for specific application scenarios (for example, an image containing NGINX or an AI software stack).

#### Base Container Images

1. Name: **openeuler/openeuler**
2. Tags: Use the openEuler version name as the tag (such as **22.03-lts**, **22.03-lts-sp1**, and **23.03**).
3. Special tags:
    - **20.03:** indicates the recommended 20.03 LTS version.
    - **latest:** indicates the latest recommended version, usually an LTS version (such as **22.03-lts-sp1**).

| Version                 | Repository                                                          | Tags          | Special Tags  |
|-------------------------|---------------------------------------------------------------------|---------------|---------------|
| openEuler 20.03 LTS     | <https://repo.openeuler.org/openEuler-20.03-LTS/docker_img/>          | 20.03-lts     |               |
| openEuler 20.03 LTS SP1 | <https://repo.openeuler.org/openEuler-20.03-LTS-SP1/docker_img/>      | 20.03-lts-sp1 |               |
| openEuler 20.03 LTS SP2 | <https://repo.openeuler.org/openEuler-20.03-LTS-SP2/docker_img/>      | 20.03-lts-sp2 |               |
| openEuler 20.03 LTS SP3 | <https://repo.openeuler.org/openEuler-20.03-LTS-SP3/docker_img/>      | 20.03-lts-sp3 |               |
| openEuler 20.03 LTS SP4 | <https://repo.openeuler.org/openEuler-20.03-LTS-SP4/docker_img/>      | 20.03-lts-sp4 | 20.03         |
| openEuler 20.09         | <https://archives.openeuler.openatom.cn/openEuler-20.09/docker_img/>  | 20.09         |               |
| openEuler 21.03         | <https://archives.openeuler.openatom.cn/openEuler-21.03/docker_img/>  | 21.03         |               |
| openEuler 21.09         | <https://archives.openeuler.openatom.cn/openEuler-21.09/docker_img/>  | 21.03         |               |
| openEuler 22.03 LTS     | <https://repo.openeuler.org/openEuler-22.03-LTS/docker_img/>          | 22.03-lts     |               |
| openEuler 22.03 LTS SP1 | <https://repo.openeuler.org/openEuler-22.03-LTS-SP1/docker_img/>      | 22.03-lts-sp1 |               |
| openEuler 22.03 LTS SP2 | <https://repo.openeuler.org/openEuler-22.03-LTS-SP2/docker_img/>      | 22.03-lts-sp2 |               |
| openEuler 22.03 LTS SP3 | <https://repo.openeuler.org/openEuler-22.03-LTS-SP3/docker_img/>      | 22.03-lts-sp3 | 22.03, latest |
| openEuler 22.09         | <https://repo.openeuler.org/openEuler-22.09/docker_img/>              | 22.09         |               |
| openEuler 23.03         | <https://repo.openeuler.org/openEuler-23.03/docker_img/>              | 23.03         |               |
| openEuler 23.09         | <https://repo.openeuler.org/openEuler-23.09/docker_img/>              | 23.09         |               |

#### Application Container Images

1. Name: **openeuler/**_{app}_, where _{app}_ is the application name.
2. Tags: The **meta.yml** file defines the application container image tags and corresponding Dockerfiles. Here's an example:
    - **tag**: The application container image tags are in the format of _\<app-version>_**-**_\<os>\<os-version>_. For example, the tag for the httpd 2.4.51 application image is **2.4.51-oe2203sp2**.
    Note: AI container image tags follow the convention outlined in [oEEP-0014](./oEEP-0014%20openEuler%20AI%20Container%20Image%20Stack%20Specifications.md) due to their complex software stacks.
    - **file**: Dockerfile used to build the image

### 2. Code Repository

<https://gitee.com/openeuler/openeuler-docker-images>

### 3. Code Contribution and Review

1. Upload: Upload the Dockerfile, **meta.yml** file, and any scripts to the openeuler-docker-images repository.

2. Review: Cloud Native SIG maintainers review the code and merge upon approval.

### 4. Release Process

#### Base Container Images

1. The Release SIG releases the **openEuler-docker.**_{arch}_**.tar.xz** file to **repo.openeuler.org** as follows:
    - **(Existing releases)**: Initial versions are released to <https://repo.openeuler.org/openEuler-{VERSION}/docker_img/>.
    - **(Existing releases)**: Update versions are released to <https://repo.openeuler.org/openEuler-{VERSION}/docker_img/update/YYYY-MM-DD/>. The latest updated version is also copied to <https://repo.openeuler.org/openEuler-{VERSION}/docker_img/update/current/>.

2. [EulerPublisher](https://gitee.com/openeuler/eulerpublisher), the one-click publishing tool maintained by the Infrastructure SIG, to retrieve the published raw container image file and publishes it to third-party container repositories.

- EulerPublisher provides the following functions:

    ```shell
    # Retrieval
    eulerpublisher container base prepare --version ${VERSION}
    # Push
    eulerpublisher container base push --version ${VERSION} --repo openeuler/openeuler
    # Test
    eulerpublisher container base check --version ${VERSION} --repo openeuler/openeuler

    # Retrieval, test, and push
    eulerpublisher container base publish --version ${VERSION} --repo openeuler/openeuler
    ```

- Currently, EulerPublisher utilizes Jenkins jobs for container image publication (for example, [**publication**](https://jenkins.osinfra.cn/job/luweijun/job/eulerpublisher/47/) of update images of the 20.03 and 22.03 LTS versions).

#### Application Container Images

Application container image Dockerfiles submitted to the [openeuler-docker-images](https://gitee.com/openeuler/openeuler-docker-images) repository are built and validated using EulerPublisher. Upon successful validation, the images are published to third-party container repositories using the following command:

```shell
# One-click publication of an application container image using EulerPublisher
eulerpublisher container app publish --repo openeuler/{APP_NAME} --tag {TAG}
```
