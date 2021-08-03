---


---

<h3 id="ci-bot-功能移植之与prow功能异同调研">CI-BOT 功能移植之与prow功能异同调研</h3>
<h4 id="cla-a-idcla-">CLA <a id="cla"></a></h4><a id="cla">

<table>
<thead>
<tr>
<th>CI-BOT</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>检查一个Pull Request的CLA状态。提供强制检测CLA状态指令</td>
<td>具备同等功能</td>
</tr>
</tbody>
</table><p><strong>处理方案：无</strong></p>
</a><h4 id="approve-a-idapprove"><a id="cla">approve </a><a id="approve"></a></h4><a id="approve">

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
<td>prow根据项目配置的owners文件，以及PR改变的文件所在目录的<br>以及父目录来选择approvers,并且当且仅当涵盖所有文件修改的<br>目录一个或多个approver执行了approve指令，该PR才会打上approve标签。<a href="https://github.com/kubernetes/test-infra/blob/master/prow/plugins/approve/approvers/README.md">文档链接</a><br>具备/reject指令，表示approver不接受该PR，PR需要修改。</td>
</tr>
</tbody>
</table><p><strong>处理方案：approve 插件prow的现有机制更具备优势，处理方案为应用当前prow的机制而不移植CI-BOT机制。</strong></p>
</a><h4 id="lgtm-a-idlgtm"><a id="approve">LGTM </a><a id="lgtm"></a></h4><a id="lgtm">

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
<td>1.所有人可以使用/lgtm指令，根据owners文件来确定对PR合入流程有效的/lgtm指令的人<br>2. 只会存在一个lgtm标签<br>3. 提供/lbtm指令（look bad to me)<br></td>
</tr>
</tbody>
</table><p><strong>处理方案：prow LGTM插件的现有机制处理方式更优，处理方案为使用当前prow的机制而不移植CI-BOT机制。</strong></p>
</a><h4 id="label标签-a-id-label-"><a id="lgtm">LABEL标签 </a><a id="label"></a></h4><a id="label">

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
</table><p>**处理方案：无 **</p>
</a><h4 id="closereopen-a-idclose_reopen"><a id="label">close&amp;reopen </a><a id="close_reopen"></a></h4><a id="close_reopen">

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
<td>lifecycle 插件为issue和PR提供了CI-BOT现有功能。</td>
</tr>
</tbody>
</table><p><strong>处理方案：无</strong></p>
</a><h4 id="pr合入-a-id-pr_merge-"><a id="close_reopen">PR合入 </a><a id="pr_merge"></a></h4><a id="pr_merge">

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
</table><p><strong>处理方案：使用tide组件。</strong></p>
</a><h4 id="assign-issue-or-pr-a-id-assign"><a id="pr_merge">ASSIGN issue or PR </a><a id="assign"></a></h4><a id="assign">

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
<td>assign插件支持为PR与issue手动触发指定责任人，</td>
</tr>
</tbody>
</table><p>**处理方案：无</p>
</a><h4 id="冻结期pr合入权限-a-idcontrol_merge_pr"><a id="assign">冻结期PR合入权限 </a><a id="control_merge_pr"></a></h4><a id="control_merge_pr">

<table>
<thead>
<tr>
<th>CI-BOT</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>希望在版本冻结期内对码云代码合入做权限控制。在仓库中配置需冻结的分支信息<br>以及冻结分支可合并的owner名字，当在使用/lgtm /approve /check-pr时触发合入PR时<br>需检测PR的基础分支是否为冻结状态，冻结状态则只有配置的owners可触发合并</td>
<td>无相关功能</td>
</tr>
</tbody>
</table><p><strong>处理方案：prow需开发并提供相应功能。</strong></p>
</a><h4 id="need-squash-a-idrebase"><a id="control_merge_pr">NEED-SQUASH </a><a id="rebase"></a></h4><a id="rebase">

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
<td>待确认</td>
</tr>
</tbody>
</table><p><strong>处理方案：需确认</strong></p>
</a><h4 id="清除pr标签-a-id-clear_label"><a id="rebase">清除PR标签 </a><a id="clear_label"></a></h4><a id="clear_label">

<table>
<thead>
<tr>
<th>CI-BOT</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>根据配置项配置要清除的标签，当PR发生source_branch_change事件时移除对应的标签</td>
<td>支持清除可配置的标签</td>
</tr>
</tbody>
</table><p><strong>处理方案：无</strong></p>
</a><h4 id="测试-a-id-retest-"><a id="clear_label">测试 </a><a id="retest"></a></h4><a id="retest">

<table>
<thead>
<tr>
<th>CI-BOT</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>/retest指令在CI-BOT源码中无相关实现</td>
<td>trigger插件会触发测试任务，并给出测试结果反馈；<br>并支持/retest指令触发测试任务</td>
</tr>
</tbody>
</table><p><strong>解决方案：已支持</strong></p>
</a><h4 id="自动添加sig标签-a-idauto_sig"><a id="retest">自动添加sig/*标签 </a><a id="auto_sig"></a></h4><a id="auto_sig">

<table>
<thead>
<tr>
<th>CI-BOT</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>在PR创建时根据仓库所属sig组添加sig/* 标签</td>
<td>无相关功能</td>
</tr>
</tbody>
</table><p><strong>处理方案：需确认</strong></p>
</a><h4 id="自动创建仓库-a-idrepohandle"><a id="auto_sig">自动创建仓库 </a><a id="repohandle"></a></h4><a id="repohandle">

<table>
<thead>
<tr>
<th>CI-BOT</th>
<th>PROW</th>
</tr>
</thead>
<tbody>
<tr>
<td>CI-BOT 会定时轮询或监听webhook事件检测openEuler/communtiy中仓库配置相关的yaml文件是否有更新，根据跟新的内容创建仓库，创建仓库分支，指定仓库所属sig组管理员。</td>
<td>无相关功能</td>
</tr>
</tbody>
</table><p>处理方案：待确认</p>
<h4 id="功能异同及开发计划">功能异同及开发计划</h4>

<table>
<thead>
<tr>
<th>功能</th>
<th>需移植/开发</th>
<th>优先级</th>
<th>计划用时</th>
</tr>
</thead>
<tbody>
<tr>
<td><a href="#cla">CLA签署</a></td>
<td>否</td>
<td>——</td>
<td>——</td>
</tr>
<tr>
<td><a href="#approve">approve</a></td>
<td>否</td>
<td>——</td>
<td>——</td>
</tr>
<tr>
<td><a href="#lgtm">LGTM</a></td>
<td>否</td>
<td>——</td>
<td>——</td>
</tr>
<tr>
<td><a href="#label">label标签</a></td>
<td>否</td>
<td></td>
<td></td>
</tr>
<tr>
<td><a href="#close_reopen">close&amp;reopen</a></td>
<td>否</td>
<td></td>
<td></td>
</tr>
<tr>
<td><a href="#pr_merge">PR合入</a></td>
<td>否</td>
<td>——</td>
<td>——</td>
</tr>
<tr>
<td><a href="#assign">ASSIGN issue or PR</a></td>
<td>否</td>
<td>——</td>
<td>——</td>
</tr>
<tr>
<td><a href="#control_merge_pr">冻结期PR合入权限</a></td>
<td>是</td>
<td></td>
<td></td>
</tr>
<tr>
<td><a href="#rebase">NEED-SQUASH</a></td>
<td>是</td>
<td></td>
<td></td>
</tr>
<tr>
<td><a href="#clear_label">清除PR标签</a></td>
<td>否</td>
<td></td>
<td></td>
</tr>
<tr>
<td><a href="#retest">测试用例</a></td>
<td>否</td>
<td>——</td>
<td>——</td>
</tr>
<tr>
<td><a href="#auto_sig">自动打sig/*标签</a></td>
<td>是</td>
<td></td>
<td></td>
</tr>
<tr>
<td><a href="#repohandle">自动创仓</a></td>
<td>是</td>
<td></td>
<td></td>
</tr>
</tbody>
</table></a>
