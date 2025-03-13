---
标题:     openEuler软件包版本监控配置
类别:     流程设计
摘要:     监控软件包上游版本
作者:     翟文杰(zwjsec at huawei.com)
状态:     初始化
编号:     oEEP-0021
创建日期: 2024-11-12
修订日期: 2025-03-10
---

## 背景
为了更好的维护和更新openEuler的软件包，软件包监控服务通过CI门禁自动监控openEuler软件包的上游软件版本和架构信息，从而解决软件包自动升级和查看软件包版本支持情况。可访问[软件中心](https://easysoftware.openeuler.org/zh/)查看软件包上游版本信息

## 软件包监控服务如何监控上游版本信息

软件包监控服务通过获取[community](https://gitee.com/openeuler/community/blob/master/sig/README.md)仓库的代码仓描述文件，解析到仓库的软件包上游信息配置，从而通过配置中的上游包链接和规则拉取到软件包的上游版本。
实现上游软件包配置方式需要在[代码仓描述文件格式](https://gitee.com/openeuler/community/blob/master/sig/README.md#%E4%BB%A3%E7%A0%81%E4%BB%93%E6%8F%8F%E8%BF%B0%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F) 中`扩充upstream字段含义`。

### 代码仓描述文件格式
upstream 清单中每个元素代表软件包上游版本信息，以关系字典的方式呈现，需要包含以下子元素:
| 名称 | 类型 | 说明 |
| :-- | :-- | :-- |
| Version URL  | (必选)字符串 | 包含软件包版本信息的url，如https://mirrors.edge.kernel.org/pub/software/network/ethtool/ 或者 https://ftp.gnu.org/gnu/gmp/ 等。
| Regex  | (必选)字符串 | 正则表达式用于从`Version URL`解析捕获出正确软件版本号 |
| homepage  | (必选)字符串 | 表示软件包主页地址，可不包含软件包版本号等信息。可将存量upstream字段中的url复制至该处 |

### 代码仓描述文件样例
```
- name: alluxio
  description: 'alluxio'
  branches:
  - name: master
    type: protected
  type: public
  upstream: 
    homepage: https://github.com/Alluxio/alluxio/
    regex: (?:Alluxio\s+)?v(\d+\.\d+\.\d+) 
    version_url: https://github.com/Alluxio/alluxio/releases 
```

`version_url`: https://github.com/Alluxio/alluxio/releases 带有alluxio软件包版本号的url连接地址

`regex`: (?:Alluxio\s+)?v(\d+\.\d+\.\d+)
1. ```(?:Alluxio\s+)?```：这是一个非捕获组（由于前面的?:），并且它是可选的（由于后面的?）。这意味着括号内的内容可以出现0次或1次。在这个例子中，它用于匹配可能存在的“Alluxio ”前缀（注意后面有一个空格）。由于是非捕获组，所以即使匹配成功，这部分内容也不会被单独捕获或保存。
2. ```Alluxio\s+```：这是非捕获组内的内容。Alluxio是字面量字符串，表示要匹配的文本中包含“Alluxio”。\s+匹配一个或多个空白字符（如空格、制表符等），确保“Alluxio”和后面的“v”版本标识符之间有空格分隔
3. ```(\d+\.\d+\.\d+)```: 这是一个捕获组，用于匹配并捕获具体的版本号。\d+匹配一个或多个数字字符，.在这里是字面量点字符（表示版本号中的分隔符），由于.在正则表达式中有特殊含义（匹配任意单个字符），所以需要使用\进行转义以表示其字面量意义。因此，\d+\.\d+\.\d+匹配形如1.2.3的版本号，其中每个数字部分至少有一位数字

这段正则表达式用于匹配形如“Alluxio 1.2.3”或“v1.2.3”的字符串，并捕获其中的版本号“1.2.3”。如果字符串包含“Alluxio ”前缀，该前缀将被忽略，只有版本号部分被捕获。


## 用户如何查看上游软件包版本

用户在向[community](https://gitee.com/openeuler/community/blob/master/sig/README.md)仓库提交PR修改代码仓描述文件（upstream信息）。后端门禁自动检测所配置的软件包字段是否能获取上游版本。

综上所述，软件包维护者需维护以下3个字段，其余字段可由后端服务自动配置。

1. name字段，表示软件包名称
2. upstream.version_url字段，表示包含软件包版本号的url
3. upstream.regex字段，表示正则表达式，配合version_url字段，从url连接中解析软件包版本号
4. upstream.homepage字段，表示软件包主页地址，不一定包含软件包版本号的url。可将存量upstream处的url地址复制粘贴至该字段

提交PR之后，用户查看软件包上游版本的途径

1. 在PR评论里看到上游软件包版本
如果CI门禁能够根据用户提交的字段查找到上游版本，返回格式如下(例子：gcc)。
```bash
Find upstream version of package: gcc.
The version field only indicates the version at the time of the this comment, maybe not the latest version.

upstream:
- version_url: https://github.com/gcc-mirror/gcc
  version: 14.2.0
```
如果门禁无法找到上游版本，返回格式如下(例子：gcc)

```bash
Fail to find upstream version of package: gcc.
```

2. 登录[软件中心](https://easysoftware.openeuler.org/zh/)的【上游软件包全景】查看

由于软件包上游url比较复杂，可能无法获取上游版本。为避免影响其他业务，后端门禁不影响合入PR流程。同时可以联系 [ @zwjsec ](https://gitee.com/zwjsec) 或者软件包的maintainer。