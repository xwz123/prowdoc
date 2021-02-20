---


---


<table>
<thead>
<tr>
<th>功能</th>
<th>功能说明</th>
<th>相关指令(使用者)</th>
<th>prow是否存在</th>
<th>需开发</th>
<th>diff prow</th>
</tr>
</thead>
<tbody>
<tr>
<td>CLA</td>
<td>检测PR的CLA状态，检测PR作者是否签署CLA并为</td>
<td>/check-cla (任何人)</td>
<td>是</td>
<td>待确定</td>
<td></td>
</tr>
<tr>
<td>LGTM</td>
<td>1.为一个Pull Request添加或者删除<code>lgtm</code>标签；<br>2.可指定lgtm标签数量，多个则添加lgtm-[login]标签并支持单独为组织或仓库进行数量配置;<br>3.添加标签的同时判断是否达到合入条件，达到则合入。</td>
<td>/lgtm  [cancel] (协作者)</td>
<td>是</td>
<td>待确定</td>
<td></td>
</tr>
<tr>
<td>APPROVE</td>
<td>为一个Pull Request添加或者删除<code>approved</code>标签，添加标签的同时判断是否达到合入条件(可配置)，达到则合入。</td>
<td>/approve [cancel] (协作者)</td>
<td>是</td>
<td>待确定</td>
<td></td>
</tr>
<tr>
<td>[remove-] kind</td>
<td>添加或者删除诸如 kind/bug 之类的标签</td>
<td>/[remove]kind     bug (任何人)</td>
<td>否</td>
<td>待确定</td>
<td></td>
</tr>
<tr>
<td>[remove-] sig</td>
<td>添加或者删除sig类型的标签 如：sig/kernel</td>
<td>/[remove-]sig kernel(任何人)</td>
<td>否</td>
<td>待确定</td>
<td></td>
</tr>
<tr>
<td>/[remove-]priority</td>
<td>添加或者删除这种priority类型的标签。 例如：<code>priority/high</code>标签。</td>
<td>/[remove-]priority high(任何人)</td>
<td>否</td>
<td>待确定</td>
<td></td>
</tr>
<tr>
<td>/close</td>
<td>关闭一个issue或者PR</td>
<td>/close (作者和协作者)</td>
<td>是（待合入 ）</td>
<td>待确定</td>
<td></td>
</tr>
<tr>
<td>/reopen</td>
<td>重新打开一个issue</td>
<td>/reopen(作者和协作者)</td>
<td>是（待合入）</td>
<td>待确定</td>
<td></td>
</tr>
<tr>
<td>/retest</td>
<td>重跑测试用例</td>
<td>/retest(任何人)</td>
<td>是</td>
<td>待确定</td>
<td></td>
</tr>
<tr>
<td>/check-pr</td>
<td>重新检测PR是否可以合入，满足条件则合入</td>
<td>/check-pr</td>
<td>是</td>
<td>待确定</td>
<td></td>
</tr>
<tr>
<td>/[un]assign</td>
<td>为issue指定/取消责任人</td>
<td>/[un]assign @login</td>
<td>否</td>
<td>带确定</td>
<td>1.ci-bot针对issue；prow针对PR</td>
</tr>
<tr>
<td>创建仓库</td>
<td>定期根据特定文件配置变化创建仓库，创建分支（是否），指定协作者。</td>
<td>无</td>
<td>是</td>
<td>待确定</td>
<td></td>
</tr>
<tr>
<td>版本冻结期代码合入权限控制</td>
<td>支持配置版本冻结的仓库，并指定特定的人可以触发相关合入指令</td>
<td>无</td>
<td>待确定</td>
<td>待确定</td>
<td></td>
</tr>
<tr>
<td>squash commit</td>
<td>可配置PRcommit的数量阙值和标签，当PR的commit数量超过阙值，为PR加上特定标签 如 stat/needs-squash</td>
<td>无</td>
<td>待确定</td>
<td>待确定</td>
<td></td>
</tr>
</tbody>
</table>
