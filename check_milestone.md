---


---

<h4 id="需求描述">需求描述</h4>
<ol>
<li>一个PR必须关联一个issue。如未关联打上相应标签，并给用户提示。</li>
<li>用户提交issue后,针对没有选择里程碑的issue,打上标签（例如：没有选择里程牌），并在对应issue下生成评论，提醒用户选择里程碑。</li>
</ol>
<h4 id="问题">问题</h4>
<ol>
<li>修改PR描述，关联issue不会触发webhook事件。</li>
<li>issue关联里程碑，不会触发webhook事件。</li>
</ol>
<h4 id="解决方案">解决方案</h4>
<p>设计associate 插件：</p>
<ol>
<li>针对需求1 ：监听PR创建的hook，如果PR未关联Issue则添加<strong>miss/issue</strong>，并添加评论：<strong>@Author Pullrequest must be associated with an issue, please associate at least one issue. after associating an issue, you can use the /check-issue command to remove the miss/issue label.</strong></li>
<li>针对需求2：监听issue的创建hook  检测是否设置里程碑，未设置则打上 <strong>miss/milestone</strong> 标签，并在评论区添加评论： <strong>@author You have not selected a milestone, please select a milestone.after setting the milestone, you can use the /check-milestone command to remove the miss/milestone label.</strong></li>
<li>针对需求1 ：提供 /check-issue 指令；检查PR是否关联issue 指令含义如下：
<ol>
<li>未关联issue，且不存在 miss-issue标签则添加 miss-issue 标签并添加评论提示；</li>
<li>已关联issue，且存在 miss-issue 标签则移除miss-issue 标签</li>
</ol>
</li>
<li>针对需求1 ： 提供/remove-miss/issue指令移除miss/issue标签（approve 和assigne可以使用） 暂时使用IsCollaborator方法判断,后面根据Owner文件实现.</li>
<li>针对需求2 ：提供 /check-milestone指令 检查issue是否关联milestone：
<ol>
<li>未关联里程碑，且不存在 miss/milestone标签则添加miss/milestone标签并添加评论；</li>
<li>已关联里程碑，且存在 miss/milestone标签则移除 miss/milestone标签；</li>
</ol>
</li>
</ol>
<p>/usr/bin/prow_review_tool -t “Merge Request Hook” -p "{“action”:“update”,“action_desc”:“source_branch_changed”,“pull_request”: {“html_url”:"<a href="https://gitee.com/openeuler/openEuler-Advisor/pulls/324%22%7D%7D%22">https://gitee.com/openeuler/openEuler-Advisor/pulls/324"}}"</a></p>

