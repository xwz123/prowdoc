---


---

<h3 id="需求描述">需求描述</h3>
<p><strong>用户提交issue后,针对没有选择里程碑的issue,打上标签（例如：没有选择里程牌），并在对应issue下生成评论，提醒用户选择里程碑。</strong></p>
<h3 id="问题">问题</h3>
<ul>
<li>gitee 关联里程碑没有hook事件，不能自动移除unset-milestone标签。</li>
</ul>
<h3 id="解决方案">解决方案</h3>
<ul>
<li>调研现有prow插件： milestone 插件目前功能为允许GitHub团队成员为issue和PR设置里程碑。</li>
<li>实现方式：
<ul>
<li>在gitee-plugins中新建milestone插件</li>
<li>监听issue的创建hook  检测是否设置里程碑，未设置则打上 <strong>unset-milestone</strong> 标签，并在评论区添加评论： <strong>@author You have not selected a milestone, please select a milestone.after setting the milestone, you can use the /check-milestone command to remove the unset-milestone label.</strong></li>
<li>提供/check-milestone 指令，手动触发检测，任何人都可以使用该指令。这里会删除标签。</li>
</ul>
</li>
<li>计划用时：1天</li>
</ul>

