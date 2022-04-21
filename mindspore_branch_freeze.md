# mindspore 版本冻结PR合入方案

### 需求：

在临近发版本的时候可以冻结该版本以控制PR的合入

### 方案：

#### 标签控制方案

**配置文件设置**

新增版本冻结控制文件：branch_freeze.yaml

```yaml
freeze_items:
- repo: mindspore/mindspore
  frozen_branchs:
  - master
  - v1.2.0
  owners:
  - xwzqmxxx
```

- repo: 冻结配置作用的仓库全路径
- frozen_branchs: 仓库中哪些分支需要冻结
- owners: 具备合入目标分支为冻结分支权限的人员列表

**标签设置**

`branch-frozen`：标识该PR目标分支以处于冻结状态，具有该标签的PR不会被Tide合入

`branch-frozen-mergeable`: 标识该PR目标分支以处于冻结状态，但是特权用户给予合入权限可以被Tide合入

以上标签在一个PR上不能并存，也就是一个目标分支为冻结状态的PR只能是以上两种标签之一

**指令设置**

| 指令                  | 描述                                                                                                               | 使用人               |
| --------------------- | ------------------------------------------------------------------------------------------------------------------ | -------------------- |
| /branch-freeze cancel | 修改PR冻结合入状态，添加 `branch-frozen-mergeable`标签，并移除 `branch-frozen`标签（如果存在）                 | 冻结配置文件中owners |
| /branch-freeze        | 修改PR冻结合入状态，添加 `branch-frozen`标签，以及添加冻结提示并移除 `branch-frozen-mergeable`标签（如果存在） | 冻结配置文件中owners |
| /check-freeze         | 手动触发检测PR冻结状态，根据冻结状态添加或移除冻结状态相关标签，主要应对版本冻结解除后无事件触发感知配置变化   | 任何人               |

**评论提示设置**

当PR添加上 `branch-frozen`标签时，添加如下评论：

```shell
The target branch of this PR has been frozen. If you want to merge, please invite the following peoples: @xwzqmxx use the `/branch-freeze cancel` command to indicate that the PR can be merged during the freezing period.
```

**冻结分支PR检测**

- PR 创建或PR 标签发生变化时
  
  首先检测PR目标分支已冻结：
  
  是：如果已经存在 `branch-frozen` 或 `branch-frozen-mergeable` 标签，则不做任何处理。如果不存在 `branch-frozen` 或 `branch-frozen-mergeable` 标签则添加 `branch-frozen` 并添加评论提示。
  
  否：如果已经存在 `branch-frozen` 或 `branch-frozen-mergeable` 标签，则移除标签。 否则不处理。
  
- 使用冻结指令时
  
  参照指令设置描述。

**PR合入控制**

依赖tide现有能力，将 `branch-frozen` 标签加入 missing_labels配置项，则PR包含该标签将不会被合入。

##### 方案优缺点

优点：对其它机器人服务功能无侵入，并能较好的满足业务需求。

缺点：对冻结配置的感知不佳，可能会造成PR目标已处于冻结状态而被合入。
