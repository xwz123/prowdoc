---


---

<h1 id="mindspore-issue-list">mindspore issue list</h1>
<h3 id="owners-文件支持控制到文件级别"><a href="https://gitee.com/mindspore/community/issues/I3W5PS?from=project-issue">owners 文件支持控制到文件级别</a></h3>
<h4 id="描述：">描述：</h4>
<p>当前OWNER文件仅支持控制该目录由哪些approve控制合入。</p>
<p>但针对编译或者第三方组件变更控制合入里，需要对requirement，setup.py等文件有严格控制合入的权限。<br>
所以期望OWNER文件新增支持对文件级别的合入控制。</p>
<h4 id="解决方案">解决方案</h4>
<p>待完善…</p>
<h3 id="issue-支持设置协助者"><a href="https://gitee.com/mindspore/community/issues/I3VO5A?from=project-issue">issue 支持设置协助者</a></h3>
<h4 id="描述">描述</h4>
<p>Can add a issue collaborators by command.</p>
<h4 id="解决方案-1">解决方案</h4>
<p>assign 插件新增 [add|rm]-collaborator 指令以支持添加和移除issue协助者。</p>
<h3 id="ci-测试过期的问题"><a href="https://gitee.com/mindspore/community/issues/I3U91A?from=project-issue">CI 测试过期的问题</a></h3>
<h4 id="描述-1">描述</h4>
<p>At present, the pull request which was merged into the repository may lead that the target branch such as master will be compiled failed. Because the CI test of pull request is not re-run before it is merged, which may be failed based on the latest target branch.</p>
<p>提议：</p>
<p>When the robot adds the label of ‘approved’, it will check the label of ‘ci-pipeline-passed’ to see whether it exists within 24 hours. If it is false, it will delete the label of <code>ci-pipeline-pass</code> and leave a comment to tell the author of pull request that he/she should retest again.</p>
<p>When the label of ‘ci-pipeline-passed’ is added again, the pull request will be send to merge pool.</p>
<h4 id="解决方案-2">解决方案</h4>
<p>待完善…</p>
<h3 id="改进-cla-签名中的机器人回复"><a href="https://gitee.com/mindspore/mindspore/issues/I3VZ5N?from=project-issue">改进 CLA 签名中的机器人回复</a></h3>
<h4 id="描述-2">描述</h4>
<p>Prompts Users that it may be due to an email problem when sign CLA failed.<br>
The git email must be the same as the CLA signing email</p>
<h4 id="解决方案-3">解决方案</h4>
<ul>
<li>优化提示</li>
<li>修改FAQ</li>
</ul>
<h3 id="can-we-offer-the-whitelist-for-repositories-creation-by-ci（husheng）"><a href="https://gitee.com/mindspore/community/issues/I3VNEF?from=project-issue">Can we offer the whitelist for repositories creation by CI（@husheng）</a></h3>
<h4 id="描述-3">描述</h4>
<p>Can we create a standard pipeline for repositories creation with a whitelist.</p>
<h3 id="offer-a-gpu-device-to-dx-sig-member-for-developing-recommendation-model（husheng）"><a href="https://gitee.com/mindspore/community/issues/I3TB1T?from=project-issue">offer a GPU device to DX SIG member for developing recommendation model（@husheng）</a></h3>
<h4 id="描述-4">描述</h4>
<p>As we want to develop a AI model for dx bot, we need a GPU device for training recommendation model.</p>

