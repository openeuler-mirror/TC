---
标题:     LLVM平行宇宙计划--基于LLVM技术栈构建oE软件包
类别:     特性变更
摘要:     通过LLVM技术栈重新构建openEuler的方案设计
作者:     赵川峰（cf-zhao）
状态:     初始化
编号:     oEEP-0003
创建日期: 2023-05-22
修订日期: 2023-05-25
---

## 动机/问题描述:
众所周知，GCC与LLVM是两个影响力最大的开源编译器工具链。目前，openEuler社区默认使用GCC作为构建软件包的编译器。

LLVM设计之初就采用了模块化架构，这种解耦架构有助于分别演进并创新，而通过统一的IR表示又将不同的模块有机的结合起来。LLVM 9.0版本之后属于Apache License，此License相比GCC的GPL License对各个商业公司来说更加友好。截至目前，LLVM社区社区贡献者已经达到2634人，2022年增加340人，涉及公司150+，周Commit数量超500+以上，并且10+年代码提交持续保持热度，LLVM峰会的活跃度远超GCC。

业界厂商（如Apple、高通、ARM、Intel等）将自身编译器已经切换到LLVM版本，并在LLVM的基础上进行竞争力的构建和向前演进。新兴的语言（Swift、Rust）也纷纷采用LLVM编译器基础设施。

综上所述，LLVM无论从技术上还是从应用项目上看，都是未来编译器的趋势。那么是否可以基于LLVM技术栈构建openEuler版本呢？

参考其他OS社区，MacOS、Android、ChromOS、OpenMandriva的系统默认编译器选择LLVM，Debian、Fedora允许软件包维护者选择GCC或LLVM构建。

我们建议，使能LLVM编译器构建更多的openEuler软件包，挑战基于LLVM技术栈完成openEuler版本发布，这个工作是平行与目前openEuler版本发布工作的，所以称为“LLVM平行宇宙计划”。

## 收益分析
* 基础性能：LLVM相对GCC更容易进行编译优化增强，具备更新的性能潜力；另外LLVM具有功能更强大的LTO能力。
* 软件包性能：软件包维护者可以选择GCC或LLVM（性能更好者）作为构建工具链，可以释放更多精力在软件功能实现上。
* 代码安全：Clang+LLVM通常对C/C++语言标准遵从更严格，通过Clang+LLVM可能发现软件包潜在缺陷；

## 方案的详细描述：
方案的总体思路是允许软件包维护者在构建软件包时选择GCC或LLVM作为构建工具链，另外整体构建工程可以选择把GCC或LLVM作为默认构建工具链。
### 设置默认编译器工具链
在`openEuler-rpm-config`软件包中加入对默认工具链的选择。
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
### 软件包构建问题修复
对所有软件包进行分析，解决构建出错问题。spec中可以使用下面条件语句进行差异化设置：
```abap
%if %{with toolchain_clang}
   xxxxxx
%endif
```
### LTO的支持
待增强

### LLVM工具链全栈支持
目前binutils和runtime仍使用GNU工具链中，后续进行LLVM全栈工具链支持。

## 影响分析
### 对软件包维护者的影响
当LLVM平行宇宙计划基线化之后，软件包维护者需保证GCC和LLVM均可以构建其软件包，除非当前软件包只支持GCC或LLVM构建。
### 对QA的影响
理论上QA需要保证GCC和LLVM构建的openEuler版本质量，当LLVM构建尚不成熟时，先基于平行宇宙计划的QA做平行保证。

### 对CICD的影响
理论上CICD需要支持GCC和LLVM构建资源，当LLVM构建尚不成熟时，先基于平行宇宙计划的资源做平行保证。

### 对release的影响
理论上需要正式发布LLVM构建的openEuler版本，当LLVN构建尚不成熟或竞争力不足时，先基于平行宇宙计划发布preview版本。

## 如何测试
软件包的维护者基于LLVM构建其软件包，然后跑通软件包的测试套。
一般情况下，openEuler社区的QA工作应该已经满足相关测试要求。

