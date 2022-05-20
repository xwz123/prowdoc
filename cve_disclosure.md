## CVE 提前披露方案

### 需求描述

在gitee 平台对 CVE issue 分析完成后，支持使用命令生成 CVE 文件，该文件支持导入CVE 官网公告平台。 

### 解决方案

#### 1. cve-manager 每天定时整理已分析完成的  CVE 并发送邮件告知安全委员会成员；
   
   流程图：

```flow
st=>start: issue 责任人对cve进行分析
op=>operation: cve-manager校验分析结果
cond=>condition: 分析完成?
oprec=>operation: 记录CVE到可披露数据表
e=>end: 结束

st->op->cond
cond(yes)->oprec
cond(no)->e
```
                    


#### 2. cve-manager 提供 `/disclosure` 指令，安全委员成员在收到邮件后，在对应CVE issue 评论中使用该指令触发生成提前披露的CVE文件并上传OBS；


#### 3.  SA 服务提供手动触发和定时任务两种方式将CVE文件导入官网；


| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |

| Item      | Value |
| --------- | -----:|
| Computer  | $1600 |
| Phone     |   $12 |
| Pipe      |    $1 |
                
----


                
### 绘制流程图 Flowchart

```flow
st=>start: 用户登陆
op=>operation: 登陆操作
cond=>condition: 登陆成功 Yes or No?
e=>end: 进入后台

st->op->cond
cond(yes)->e
cond(no)->op
```
                    
### 绘制序列图 Sequence Diagram
                    
```seq
Andrew->China: Says Hello 
Note right of China: China thinks\nabout it 
China-->Andrew: How are you? 
Andrew->>China: I am good thanks!
```

### End