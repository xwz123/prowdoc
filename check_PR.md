---


---

<h4 id="需求：">需求：</h4>
<ol>
<li>移植CI-BOT PR创建和PR源代码改变检查commit数量</li>
<li>版本冻结期代码合入权限控制。
<ul>
<li>背景：对于openEuler 假设20.03-LTS版本近期需要发布一个release版本，管理员在openEuler/release-management/release-management.yaml下设置了对openEuler组织下所有仓库的20.03-LTS分支进行冻结，并配置在冻结期内可以具有合入权限的owners人员列表，则只有这些人员具备合入PR的权限。</li>
</ul>
</li>
</ol>
<h4 id="解决方案">解决方案</h4>
<p><code>新增checkpr插件：</code></p>
<ul>
<li>针对需求1：<br>
移植CI-BOT现有逻辑，当PR创建或发生源分支改变是判断commit数量，超过指定的阈值，打上stat/need-squash</li>
<li>针对需求2：<br>
结合tide以标签来控制PR在冻结期的合入，若PR的base branch 是冻结期内，则打上stat/version-freeze,并提供 /remove-stat/version-freeze指令，该指令的只能被冻结期合入配置的owners使用。</li>
</ul>

