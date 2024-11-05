---
Title:          openEuler AI Container Image Stack Specifications
Type:           Process
Abstract:       Layering and tagging specifications for openEuler AI container image software stacks
Author:         Lu Weijun <wjunlu217@gmail.com>
Status:         Initial
No:             oEEP-0014
Create_time:    2023-10-20
Update_time:    2023-10-20
---

## Motivation and Objectives

Diverse AI acceleration libraries and SDKs offered by vendors like Nvidia, AMD, and Ascend pose challenges for users deploying AI applications across different hardware and software environments. To address this, vendors have introduced AI container images tailored to their hardware. These images simplify deployment, allowing users to quickly launch pre-configured environments, saving time and effort.

Against this backdrop, the demand for AI container images is growing rapidly, but developers lack clear guidance on their creation and best practices. Key questions remain unanswered: What software should be included in images for specific use cases? How should image tags be standardized for clarity and ease of use?

### Current Status

The following table illustrates the software stack and tag examples of AI container images from leading vendors.
|           | Nvidia    | AMD     |
|-----------|------------|---------|
| Software stack| **base**: CUDA runtime (cudart) <br>**runtime**: **base** + CUDA math libraries + NCCL + cuDNN <br>**devel**: **runtime** + tools for building CUDA images | ROCm + pytorch <br> ROCm + tensorflow |
| Tags      | `<cuda-version>-base/runtime/devel-<os><os-vsesion>` <br>Example: `12.2.0-runtime-ubuntu20.04` | `rocm<rocm-version>_<os><os-version>_py<py-version>_pytorch_<pytorch-version>`  <br> `rocm<rocm-version>-<os><os-version>-tf<tf-version>-dev` <br>Example: `rocm5.7_ubuntu22.04_py3.10_pytorch_2.0.1`  |

1. Software stacks

    - Nvidia's **base** image (available at <https://hub.docker.com/r/nvidia/cuda>) includes CUDA and cuDNN from <https://gitlab.com/nvidia/container-images/cuda>). The **runtime** image includes nvidia-container-runtime from <https://github.com/NVIDIA/nvidia-container-runtime>.
    - AMD's images (available at <https://hub.docker.com/r/rocm/pytorch> and <https://hub.docker.com/r/rocm/tensorflow>) include ROCm (<https://github.com/RadeonOpenCompute/ROCm>), PyTorch or TensorFlow, and the corresponding Python version.

    Despite variations in pre-installed software, the software stack layering is generally consistent. Including large language models (LLMs), AI container images can be categorized into the following layers:

    ```mermaid
    graph LR;
    A[(<br> LLMs <br> Tools <br> AI framework <br> SDK)]
    ```

2. Image tags

    Both Nvidia and AMD use image tags that combine pre-installed software and version information, typically formatted as `<sdk><sdk-version>-<os><os-version>-<framework><framework-version>`.

### Purpose of this oEEP

This proposal provides guidance to developers on building openEuler-based AI container images for various hardware platforms.

## Solution Description

### Image Content and Tagging Rules

- Base SDK image: contains the base image with the required SDK for specific hardware. The tag format is `<sdk><sdk-version>-oe<openeuler-version>`. Example: `cann7.0.RC1.alpha002-oe2203sp2`
- AI framework image: built upon the base SDK image and includes the AI framework. The tag format is `<framework><framework-version>-<sdk><sdk-version>-oe<oe-version>`. Example: `pytorch2.1.0-cann7.0.RC1.alpha002-oe2203sp2`
- LLM application image: extends the AI framework image and includes the AI application. The tag format is `<LLMs>-<framework><framework-version>-<sdk><sdk-version>-oe<oe-version>`. For example, `chatglm6b-cann7.0.RC1.alpha002-pytorch2.1.0-oe2203sp2` represents a container image based on openEuler 22.03 LTS SP2 that deploys the chatglm-6b LLM and includes cann-7.0.RC1.alpha002 and PyTorch 2.1.0.

Container images intended for development environments append `-dev` to the end of their tags, for example: `pytorch2.1.0-cann7.0.RC1.alpha002-oe2203sp2-dev`.

### Image Release (Referencing [oEEP-0005](<./oEEP-0005 openEuler Official Container Image Release Process.md>))

- AI container image builds rely on a **meta.yml** file where each key-value pair specifies the image tag and the corresponding Dockerfile.
- EulerPublisher builds and publishes the AI container images to the designated repository based on the **meta.yml** file.
