---
Title:          LLVM Parallel Universe Program: Building openEuler Software Packages with the LLVM Technology Stack
Type:           Standard
Abstract:       LLVM technology stack-based openEuler build proposal
Author:         Zhao Chuanfeng
Status:         Initial
No:             oEEP-0003
Create_time:    2023-05-22
Update_time:    2023-05-25
---

## Motivation/Problem

GCC and LLVM are the two most influential open source compiler toolchains. Currently, the openEuler community uses GCC as the default compiler for building software packages.

LLVM was designed with a modular architecture from the ground up. This decoupled architecture facilitates independent evolution and innovation, while the unified intermediate representation (IR) organically combines different modules. LLVM 9.0 and later versions are licensed under the Apache License, which is more commercially friendly than the GPL License used by GCC. As of today, the LLVM community has over 2634 contributors, with 340 added in 2022, representing over 150 companies. With over 500 commits per week and sustained code contribution for over 10 years, the LLVM community is far more active than the GCC community.

Industry vendors (such as Apple, Qualcomm, ARM, and Intel) have switched their compilers to LLVM and are building upon it to enhance competitiveness. Emerging languages (such as Swift and Rust) have also adopted the LLVM compiler infrastructure.

Therefore, LLVM is the future of compilers, both technically and in terms of application projects. So, is it possible to build openEuler using the LLVM technology stack?

Looking at other OS communities, MacOS, Android, ChromOS, and OpenMandriva use LLVM as their default system compiler, while Debian and Fedora allow package maintainers to choose between GCC and LLVM.

We propose to use LLVM to build more openEuler software packages and to explore the feasibility of building an openEuler release using entirely the LLVM technology stack. This effort will run in parallel with the current openEuler release process, hence the name "LLVM Parallel Universe Program."

## Benefits

* Base performance: LLVM is generally easier to optimize for compilation than GCC, offering greater performance potential. Additionally, LLVM boasts a more powerful link-time optimization (LTO) capability.
* Software package performance: Package maintainers can choose between GCC and LLVM (whichever performs better) as their build toolchain, allowing them to focus more on software functionality.
* Code security: Clang+LLVM is typically more strict in adhering to C/C++ language standards, potentially uncovering hidden defects in software packages.

## Solution Description

The overall approach is to allow package maintainers to choose between GCC and LLVM as the toolchain for building software packages. Additionally, the overall build process can use either GCC or LLVM as the default build toolchain.

### Setting the Default Compiler Toolchain

Add the option to select the default toolchain in the openEuler-rpm-config package.

```abap
# GCC toolchain
%__cc_gcc gcc
%__cxx_gcc g++
%__cpp_gcc gcc -E

# Clang toolchain
%__cc_clang clang
%__cxx_clang clang++
%__cpp_clang clang-cpp

# Default to the GCC toolchain
%toolchain gcc

CC="${CC:-%{__cc}}" ; export CC
CXX="${CXX:-%{__cxx}}" ; export CXX
```

### Software Package Build Issue Resolution

Analyze all software packages and resolve any build errors. The following conditional statement can be used in the spec file for differentiated settings:

```abap
%if %{with toolchain_clang}
   xxxxxx
%endif
```

### LTO Support

To be enhanced.

### Full LLVM Toolchain Support

Currently, binutils and runtime still use the GNU toolchain. Full LLVM toolchain support will be added in the future.

## Impacts

### Impacts on Package Maintainers

Once the LLVM Parallel Universe Program is baselined, package maintainers will need to ensure that their packages can be built with both GCC and LLVM, unless the package only supports GCC or LLVM building.

### Impacts on QA

Theoretically, quality assurance (QA) needs to ensure the quality of openEuler versions built by both GCC and LLVM. When LLVM builds are not yet mature, parallel QA will be conducted based on the LLVM Parallel Universe Program.

### Impacts on CI/CD

Theoretically, continuous integration (CI)/continuous delivery (CD) needs to support build resources for both GCC and LLVM. When LLVM builds are not yet mature, parallel support will be provided based on the resources of the LLVM Parallel Universe Program.

### Impacts on Releases

Theoretically, LLVM-built openEuler versions will be officially released. When LLVM builds are not mature or competitive enough, preview versions will be released based on the LLVM Parallel Universe Program.

## How to Test

Package maintainers build their packages based on LLVM and then run the test suite for the package.
Under normal circumstances, QA of the openEuler community should already meet the relevant testing requirements.
