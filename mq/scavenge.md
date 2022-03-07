## Task Description

当前社区的PR存在较多长时间未合入未关闭情况，对于PR来说比较清楚，一般都是在一定时间内必须合的，基本上不存在长时间不合入需要挂起的情况。

## Task Goal

希望过程有一个及时合入提醒（每个月1次），超过一段时间后给出关闭说明并进行关闭操作。

#### 处理方式

实现robot-gitee-scavenger机器人，该机器人为一个可配置的周期任务：

 ![](C:\otherProject\prowdoc\mq\images\clear_pr.png)

test case:

https://gitee.com/cve-manage-test/cbdd/pulls/2#note_8698910

我们开发了scavenger bot 来实现上述需求：

> 1. scavenger 每天会检查一次配置的需要检查的组织或仓库中的所有PR；
> 2. 根据提醒时间间隔设置，当PR的创建时间或者上一次合入提醒距离当前时间的间隔大于该设置将添加评论提醒合入；
>   - [合入提醒案例一](https://gitee.com/cve-manage-test/whtest9/pulls/1#note_8700508)
>   - [合入提醒案例二](https://gitee.com/cve-manage-test/whtest9/pulls/1#note_8700508)
> 3. 根据一个PR的最大开启时间设置，当PR创建时间达到该配置时间将给出提醒，并关闭PR;
>   - [关闭案例](https://gitee.com/cve-manage-test/cbdd/pulls/2#note_8698910)

[code review 连接](https://github.com/opensourceways/robot-gitee-scavenger/pull/3)

**PS: 请根据测试效果以及实现代码判断是否达到需求预期，并给出您的宝贵建议！**

