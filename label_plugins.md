---


---

<h3 id="label插件">label插件</h3>
<h4 id="需求">需求</h4>
<ol>
<li>对CI-BOT [remove](kind|sig|priority）指令功能进行移植。</li>
<li>限制PR作者通过码云界面为PR添加重要标签。</li>
<li>当PR源代码发生变化，需清除一些重要标签。</li>
</ol>
<h4 id="功能实现">功能实现</h4>
<h5 id="针对需求1（已完成）：为issue或pr提供如下指令添加或移除标签：">针对需求1（已完成）：为issue或PR提供如下指令添加或移除标签：</h5>

<table>
<thead>
<tr>
<th>指令</th>
<th>功能</th>
</tr>
</thead>
<tbody>
<tr>
<td>/[remove-]kind (如：/kind bug /remove-kind bug)</td>
<td>任何人都可以使用该指令为issue加上或移除kind/bug标签</td>
</tr>
<tr>
<td>/[remove-](area|committee|kind|language|priority|sig|triage|wg)</td>
<td>同kind指令</td>
</tr>
<tr>
<td>/[remove-]label</td>
<td>添加标签指令,在上述指令上额外提供的打自定义标签的功能，需在label的配置文件中配置标签。（如：用户需要使用euler-666标签，可以在配置文件中additional_labels字段下配置 euler-666 就可以使用 [remove-]label euler-666指令添加或移除标签）</td>
</tr>
</tbody>
</table><p>** 注意：如需这些指令生效，issue或PR所在的仓库必须已配置有对应的标签**</p>
<h5 id="针对需求2：新增配置项配置pr作者自己不能通过码云界面添加的标签，当新建pr或标签变动时检查pr现有的标签是否在配置项中，存在就获取pr的操作日志检查标签的添加者，如添加者为作者就移除该标签。">针对需求2：新增配置项配置PR作者自己不能通过码云界面添加的标签，当新建PR或标签变动时检查PR现有的标签是否在配置项中，存在就获取PR的操作日志检查标签的添加者，如添加者为作者就移除该标签。</h5>
<h5 id="针对需求3：新增配置项配置当pr发生source_branch_change事件时要清除的标签，当事件发生时根据配置和pr已存在的标签移除对应的标签。">针对需求3：新增配置项配置当PR发生source_branch_change事件时要清除的标签，当事件发生时根据配置和PR已存在的标签移除对应的标签。</h5>

