---
标题:     openEuler新增branch-keeper角色作为仓库的分支守护者
类别:     流程设计
摘要:     社区成员权限管理
作者:     曹志 (george at openeuler.sh)
状态:     初始化
编号:     oEEP-0016
创建日期: 2024-02-07
修订日期: 2024-02-07
---

## 动机/问题描述:
### 背景
当前openEuler社区SIG组成员分三种角色，分别是maintainer、committer、admin；
maintainer作为SIG组中坚力量，关注SIG组整体健康发展，对SIG组所有仓库代码均有审核合入权限；
committer是SIG组内单个仓库代码质量看护人，有对应仓库的代码审批、合入权限；
admin是一种特殊的committer，除committer权限外，还有开启wiki、仓库同步、构建网页等部分特定功能权限；
上述三种角色的权限粒度都在仓库一层，不能细分到版本分支；
### 本oEEP解决的问题
近日部分SIG组反馈，一些仓库因committer成员较多、较分散，出现代码合入要求与版本分支管理要求不一致情况；
如kernel仓库，其仓库committer成员超70人，社区Release版本管理要求全员覆盖相对困难；
因此希望SIG组内有‘分支看护者角色’，配合committer和maintainer做好版本分支管理；
本oEEP提供一种分支看护者实现方案；

## 方案的详细描述:

### 1. 新增分支守护者
针对上述场景，基础设施与相关SIG组讨论后，提出对仓库分支新增配置**分支守护者**（branch-keeper）角色：
 1. branch-keeper提名来自仓库committer； 
 2. 通常一个仓库分支仅设置一位branch-keeper；
 3. branch-keeper配置时需要指定仓库和分支； 
 4. branch-keeper配置信息记录在sig_info.yaml中； 
 5. 仓库可以根据各自需求选择设置或不设置branch-keeper；
 6. 仓库不设施branch-keeper角色时，仓库各分支权限保持不变，授权maintainer和所有committer；
 7. 仓库设施branch-keeper后，仓库对应分支合入权限仅授权branck-keeper和maintainer；

### 2. 变更前后approve逻辑变化
变更前（或仓库未配置branch-keeper），仓库所有分支的PR合入权限：
| 角色 | Pull Request合入权限  |
|--|--|
| maintainer |  &#10004; |
| committer  |  &#10004; |

变更后，如果配置有branch-keeper的分支是PR的目标分支，该PR合入权限：

| 角色 | Pull Request合入权限  |
|--|--|
| maintainer |  &#10004; |
| committer  |  &#10005; |
| branch-keeper  |  &#10004; |

### 3. 配置方式
在[community仓库相关SIG组](https://gitee.com/openeuler/community/tree/master/sig)的sig-info.yaml文件中增加branches配置，其中sig-info.yaml的相关规范与说明参考[格式化文档]；(https://gitee.com/openeuler/community/tree/master/sig#sig%E7%9B%B8%E5%85%B3%E4%BF%A1%E6%81%AF%E7%9A%84%E6%A0%BC%E5%BC%8F%E5%8C%96%E5%AD%98%E5%82%A8)
新增branches列表中每一条记录为一组分支守护者信息，如下图所示：
~~~yaml
……
branches:
  - repo_branch:
      - repo: openeuler/kernel
        branch: openEuler-20.03-LTS-SP1
      - repo: openeuler/kernel-doc
        branch: openEuler-20.03-LTS-SP1
    keeper:
      - gitee_id: zhangsan1
	    name: zhangsan
	    organization: ABC
	    email: zhangsan***@qq.com
  - repo_branch:
      - repo: src-openeuler/kernel
        branch: openEuler-22.03-LTS-SP2
    keeper:
      - gitee_id: lisi1
        name: lisi
        organization: AABB
        email: lisi****@163.com
……
~~~
相关字段说明:
|字段  | 类型 |  说明|
|--|--|--|
| repo_branch| 列表 |一组仓库、分支信息，这一组信息配置相同的分支守护者|
| keeper| 字典 |描述分支守护者相关信息,包括gitee_id,name,organizaion,email等|



### 4. 实现流程
  ```mermaid
  graph LR;
  A(PR初始化) --> J[评论/approve];
  J[评论/approve] --> B[获取sig_info内容];
  B[获取sig_info内容] --> C{判断仓库分支是否配置branch-keeper?};
  C{判断仓库分支是否配置branch-keeper?} -- 是 --> D{评论者是否为branch-keeper或maintainer};
  C{判断仓库分支是否配置branch-keeper?} -- 否 -->  E{评论者是否为maintainer或committer};
  D{评论者是否为branch-keeper或maintainer} -- 是 --> F[获取授权PR获取approve合入标签];
  D{评论者是否为branch-keeper或maintainer} -- 否 --> G[授权失败,无合入权限];
  E{评论者是否为maintainer或committer} -- 是 --> F[获取授权PR获取approve合入标签];
  E{评论者是否为maintainer或committer} -- 否 --> G[授权失败,无合入权限];
  F[获取授权PR获取approve合入标签]-->H[结束];
  G[授权失败,无合入权限]-->H[结束];
  ```

### 5.影响评估
该变更为增量修改：
1. 对没有“分支守护者”诉求的SIG组不产生影响；
2. 对配置有“branch-keeper”的仓库和分支，将影响其作为PR目标分支的代码合入权限；
3. SIG组中部分仓库配置“branch-keeper”，对未配置的仓库和分支无影响；