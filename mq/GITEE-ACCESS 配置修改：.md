## GITEE-ACCESS 配置修改：

修改前：

继承有bug：重复发送事件到同一插件

  ![](C:\access_jichen_bug.png)

```yaml
access:
  repo_plugins:
     cve-manage-test:
       - welcome
       - approve
       - associate
     cve-manage-test/config:
       - welcome
     cve-manage-test/whtest9:
       - welcome
  plugins:
    - name: approve
      endpoint: http://192.168.1.195:8889/gitee-hook
      events:
        - "Note Hook"
        - "Issue Hook"
        - "Merge Request Hook"
     - name: welcome
      endpoint: http://192.168.1.195:8888/gitee-hook
      events:
        - "Note Hook"
        - "Issue Hook"
        - "Merge Request Hook"
    - name: associate
      endpoint: http://192.168.1.195:8899/gitee-hook
      events:
        - "Note Hook"
        - "Issue Hook"
        - "Merge Request Hook"
```

修改后：

1. 保留继承关系
2. 支持排除对应仓库的继承关系

实现方式：

1. 根据配置生成demux结构体对象作为缓存：

   ```go
   type demux struct {
   	reposEventDemux         map[string]eventsDemux // 每个仓库需要转发的事件以及endpoint，此结构中包含需继承组织所有插件的信息且不排除需要排除的仓库
   	excludeReposEventsDemux map[string]eventsDemux // 每个仓库不应该转发的事件以及endpoint
   }
   ```

2. 暴露方法在事件到达时根据org/repo过滤出正真需要转发的事件：

   ```go
   func (d demux) getEventsDemux(org, repo string) eventsDemux {
       ...
       return eventsDemux{}
   }
   ```

   

```yaml
access:
  repo_plugins:
    - repos:
       - cve-manage-test
      excluded_repos:
       - cve-manage-test/cbdd
      plugins:
       - welcome
       - associate
    - repos:
       - cve-manage-test/cbdd
      plugins:
       - approve
  plugins:
    - name: approve
      endpoint: http://192.168.1.195:8889/gitee-hook
      events:
        - "Note Hook"
        - "Issue Hook"
        - "Merge Request Hook"
    - name: welcome
      endpoint: http://192.168.1.195:8888/gitee-hook
      events:
        - "Note Hook"
        - "Issue Hook"
        - "Merge Request Hook"
    - name: associate
      endpoint: http://192.168.1.195:8899/gitee-hook
      events:
        - "Note Hook"
        - "Issue Hook"
        - "Merge Request Hook"
        
------根据以上配置运行测试的结果如下并符合预期------
cve-manage-test/cbdd:
 Issue Hook:map[http://192.168.1.195:8889/gitee-hook:{}]
 Merge Request Hook:map[http://192.168.1.195:8889/gitee-hook:{}] 
 Note Hook:map[http://192.168.1.195:8889/gitee-hook:{}]]
cve-manage-test/config:
 Issue Hook:map[http://192.168.1.195:8888/gitee-hook:{} http://192.168.1.195:8899/gitee-hook:{}] 
 Merge Request Hook:map[http://192.168.1.195:8888/gitee-hook:{} http://192.168.1.195:8899/gitee-hook:{}] 
 Note Hook:map[http://192.168.1.195:8888/gitee-hook:{} http://192.168.1.195:8899/gitee-hook:{}]]

    
```

