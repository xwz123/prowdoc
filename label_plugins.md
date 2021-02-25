---


---

<h3 id="label插件">label插件</h3>
<h4 id="功能介绍">功能介绍</h4>
<p>为issue或PR提供如下指令添加或移除标签：</p>

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
<td>同sig指令</td>
</tr>
<tr>
<td>/[remove-]label</td>
<td>添加标签指令,在上述指令上额外提供的打自定义标签的功能，需在label的配置文件中配置标签。（如：用户需要使用euler-666标签，可以在配置文件中additional_labels字段下配置 euler-666 就可以使用 [remove-]label euler-666指令添加或移除标签）</td>
</tr>
</tbody>
</table><p>** 注意：如需这些指令生效，issue或PR所在的仓库必须已配置有对应的标签。**</p>

