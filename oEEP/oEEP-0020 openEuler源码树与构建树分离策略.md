---
标题:     openEuler 源码树与构建树分离策略
类别:     特性变更
摘要:     源码树与构建树在构建过程中的分离变化
作者:     Funda Wang (fundawang at yeah.net)
状态:     活跃
编号:     oEEP-0020
创建日期: 2024-10-23
修订日期: 2025-01-25
---

# 背景说明
源码树(Source tree)，指软件包 SOURCES 展开后的源代码目录树；构建树(Build tree)，指软件进行编译过程中由源代码通过编译、转换生成的目标文件所在的目录树。由于软件包及其口味的日渐复杂，越来越多的软件包及构建系统对发行版与二进制包分发者提出建议，在构建时使用独立于源码树的构建树进行构建（Out of source build），主要原因是避免在构建过程中污染源代码，避免异构或不同口味的目标文件互相污染。

# cmake 构建系统

### 生效版本 25.03
`%cmake` 宏定义发生重大变化，构建树与源码树分离。构建树默认为源码树下的 `%{__cmake_builddir}`(默认等同于`%{_target_platform}`)文件夹。

使用 cmake 作为构建系统，且在 spec 文件中使用`%cmake`宏的开发者，应该清楚明白的知道，软件包构建过程**生成**的所有文件，包括但不限于目标文件、可执行文件、共享库、文档文件、最终配置文件范本，均不直接存储于源码树下，而在`%{__cmake_builddir}`文件夹下。如果要在后续的`%install`、`%check`、`%files`等区段引用构建过程中生成的文件，需要引用`%{__cmake_builddir}`文件夹。

特别需要指出的是，cmake 支持多种不同的编译规则生成器，比如`Unix Makefiles`和`Ninja`。无论你使用何种生成器，都建议在后续继续使用`%cmake_build`进行构建，使用`%cmake_install`进行安装，使用`%ctest`进行测试，cmake 本身会处理不同生成器之间的差异。

例1：24.03 LTS 下某软件的SPEC为：
```
%build
%cmake -Dfoo=bar
%make_build

%install
%make_install

%check
ctest

%files
%doc SOME_BUILT_FILES
```
在 25.03 及其之后的版本应修改其内容为：
```
%build
%cmake -Dfoo=bar
%cmake_build

%install
%cmake_install

%check
%ctest

%files
%doc %{__cmake_builddir}/SOME_BUILT_FILES
```

例2：24.03 LTS 下某软件的SPEC为：
```
%build
mkdir -p build && cd build
%cmake .. -GNinja
%ninja_build

%install
pushd build
%ninja_install
popd
```
在 25.03 之后应修改其内容为：
```
%build
%cmake -GNinja
%cmake_build

%install
%cmake_install
```

**禁用说明**

如果软件不支持源码树外构建，且与上游沟通无果，可由软件包自行设置在源码树中直接进行构建（in source build）。方法是定义：
```
%global __cmake_in_source_build 1
```
其他不变，仍然使用`%cmake`、`%cmake_build`、`%cmake_install`进行SPEC编写。

**参考链接**

https://fedoraproject.org/wiki/Changes/CMake_to_do_out-of-source_builds
https://cmake.org/cmake/help/latest/manual/cmake.1.html#introduction-to-cmake-buildsystems

# Meson 构建系统
`meson`构建系统的构建树与目录树不同，构建树目录名为`%{_vpath_builddir}`。
```
%build
%meson
%meson_build

%install
%meson_install

%check
%meson_test
```

# Autotools 构建系统
这一构建系统较老，由于改造困难其他发行版均没有设置单独的构建树目录。openEuler 目前采用的策略与其它发行版相同。
