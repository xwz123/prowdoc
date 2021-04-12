---


---

<h3 id="mindspore--迁入prow-功能概述">MindSpore  迁入Prow 功能概述</h3>
<h4 id="cla">CLA</h4>
<p>已使用prow CLA</p>
<h4 id="lgtm">lgtm</h4>

<table>
<thead>
<tr>
<th>CI-BOT</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>1.为一个Pull Request添加或者删除<code>lgtm</code>标签；<br>2.可指定lgtm标签数量，多个则添加lgtm-[login]标签并支持单独为组织或仓库进行数量配置;<br>3.添加标签的同时判断是否达到合入条件，达到则合入。</td>
<td>1.根据owners文件来确定具备使用/lgtm指令的人（review &amp; approve）<br>2. 只会存在一个lgtm标签，并在打上标签的同时在评论区给出lgtm标签对应的commit的hash<br>3. 具备/lgtm cancel指令取消lgtm标签<br>4. PR source_branch_change 事件会自动移除lgtm标签</td>
</tr>
</tbody>
</table><h4 id="approve">approve</h4>

<table>
<thead>
<tr>
<th>CI-BOT</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>1. 根据触发指令的人在仓库的权限执行：admin ,write<br>2. 获取owners文件，触发评论者在owner文件中即具备执行条件<br>3. 如果为community仓库则需判断PR修改的文件是否包含sig/开头的文件夹 <br>并获取该path下owners文件中是否存在评论者存则具备执行条件<br>4.隐含触发合入PR功能</td>
<td>prow根据项目配置的owners文件，以及PR改变的文件所在目录<br>以及父目录来选择approvers,并且当且仅当涵盖所有文件修改的<br>目录一个或多个approver执行了approve指令，该PR才会打上approve标签。<a href="https://github.com/kubernetes/test-infra/blob/master/prow/plugins/approve/approvers/README.md">文档链接</a></td>
</tr>
</tbody>
</table><h4 id="label标签">LABEL标签</h4>

<table>
<thead>
<tr>
<th>CI-BOT</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>1.支持如/kind bug之类的分类指令并打上如 kind/bug标签<br>2.支持如/sig kernel之类的sig分组指令并打上如 sig/kernel标签<br>3.支持如/priority hight之类的优先级标签并打上 priority/hight标签<br>4.支持/remove-[sig|kind|proiority] *指令移除对应标签<br>5.任何人在PR或issue上都可以使用上述指令</td>
<td>label插件包含BOT所有功能，且使用方式和结果于CI-BOT一致。</td>
</tr>
</tbody>
</table><h4 id="closereopen">close&amp;reopen</h4>

<table>
<thead>
<tr>
<th>CI-BOT</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>1.为仓库的协作者提供/close指令关闭issue或者PR<br>2.为作者和协作者提供/reopen指令重新开启一个已关闭的issue</td>
<td>lifecycle 插件为issue和PR提供了CI-BOT现有功能。<br>（此外还提供了生命周期标签以及指令，并将提供配置job跟踪PR和issue的生命周期自动打上对应生命周期标签，并关闭超时的issue或PR ）</td>
</tr>
</tbody>
</table><h4 id="pr合入">PR合入</h4>

<table>
<thead>
<tr>
<th>CI-BOT</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>在打上lgtm、approve标签以及手动使用checkPR时会<br>尝试检测PR合入的条件 是否具备对应标签等 ；合入或给出失败提示</td>
<td>tide 组件会定期检测PR 根据状态触发测试或合并PR,并提供合并的方式配置。</td>
</tr>
</tbody>
</table><h4 id="ci测试">CI测试</h4>

<table>
<thead>
<tr>
<th>mindSpore</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>ci-pipeline</td>
<td>trigger+prow job + jobReport</td>
</tr>
</tbody>
</table><p>/retest （与trigger 插件 /retest指令冲突）</p>
<h4 id="检查是否指定reviewer">检查是否指定reviewer</h4>

<table>
<thead>
<tr>
<th>CI-BOT</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>PR创建会检测该PR是否指定reviewer,未指定会在评论区给出提示。<br>e.g: ***<a href="https://gitee.com/mind_spore/dashboard/members/zhao_ting_v">@zhaoting </a>***Please assign one reviewer at least, visit <a href="https://gitee.com/mindspore/community/tree/master/sigs">https://gitee.com/mindspore/community/tree/master/sigs</a> to check all sigs and maintainers in community.</td>
<td>暂不支持(有替代方案)</td>
</tr>
</tbody>
</table><p>如果需要，prow可以修改 Blunderbuss插件以支持gitee平台，该插件将根据PR改动以及owners文件寻找认为最合适的2个reviewer。</p>
<h4 id="assign-issue-or-pr">ASSIGN issue or PR</h4>

<table>
<thead>
<tr>
<th>CI-BOT</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>支持为issue指定/取消责任人 /[un]assign @xxxx指定责任人</td>
<td>assign插件支持</td>
</tr>
</tbody>
</table><h4 id="need-squash">NEED-SQUASH</h4>

<table>
<thead>
<tr>
<th>CI-BOT</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>检测PR commit的数量，判断配置的阈值是否小于commit的数量，符合条件则打上 stat/need-squash</td>
<td>无对应功能</td>
</tr>
</tbody>
</table><h4 id="手动指定的很多标签（kind-comp-stat-wg）">手动指定的很多标签（kind/* comp/* stat/* wg/*）</h4>
<p>CI-BOT 有提供/kind指令<br>
comp stat wg 等标签可以在label插件中予以支持.</p>
<h4 id="pr-body中的指令含义">PR BODY中的指令含义?</h4>
<p>e.g:</p>
<p><strong>What type of PR is this?</strong></p>
<blockquote>
<p>Uncomment only one <code>/kind &lt;&gt;</code> line, hit enter to put that in a new line, and remove leading whitespaces from that line:</p>
<p>/kind bug<br>
/kind task<br>
/kind feature</p>
</blockquote>
<p><strong>CI-BOT &amp;&amp; prow 目前不支持解析PR Body中的指令</strong></p>

