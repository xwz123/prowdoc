---


---

<h4 id="需求描述">需求描述</h4>
<ol>
<li><strong>issue 定时清理</strong>
<ul>
<li>issue 必须指定人，未指定人CIBOT需及时给出提醒。</li>
<li>提醒后，指定给自己，或指定给committer和Reviewer，不知道的情况下指定给cibot进一步确认。</li>
<li>issue每隔半个月做一次提醒，及时刷新，半年内未处理则直接关闭，可以给出提示如有需要可以reopen.</li>
</ul>
</li>
<li><strong>PR 定时清理</strong>
<ul>
<li>PR必须指定人review，如不知道指定谁，可以根据代码维护目录指定reviewer。</li>
<li>要监控PR的review情况，超过一周未刷新，则请求review，发送邮件。</li>
<li>超过一个月未review，则直接关闭PR。</li>
</ul>
</li>
</ol>
<h3 id="调研k8s社区">调研K8S社区</h3>
<h4 id="调研pr">调研PR</h4>
<h5 id="pr从提交到合并的过程如下：">PR从提交到合并的过程如下：</h5>
<ol>
<li>创建PR</li>
<li>@ k8s-ci-robot分配审阅者</li>
<li>如果您不是组织的成员，那么审阅者或者 Kubernetes成员将检查拉请求是否可以安全测试。如果是这样，他们评论 /ok-to-test。组织的成员创建的PR不需要此步骤。经过此步骤后PR将被视为可信任的并运行提交前测试。
<ul>
<li>自动运行测试</li>
<li>如果测试失败，请通过将编辑推送到PR分支来解决问题</li>
<li>If the failure is a flake, anyone on trusted pull requests can comment <code>/retest</code> to rerun failed tests</li>
</ul>
</li>
<li>审核者编辑建议</li>
<li>将编辑信息推送到您的PR分支</li>
<li>根据需要重复前两个步骤，直到审阅者添加 /lgtm，/ lgtm标签在相应项目的OWNERS文件中被列为审阅者的人使用时，表示该代码已通过该项目的一名或多名可信审阅者的审阅。</li>
<li>（可选）一些审阅者希望您在此步骤中压扁提交</li>
<li>按照机器人的建议分配一个所有者，该所有者将/ approve标签添加到拉取请求中。表示代码已通过最终审核，可以自动合并。</li>
</ol>
<h5 id="更多关于ok-to-test">更多关于ok-to-test</h5>
<ul>
<li>组织成员将OK-to-test标签应用到外部贡献者的PR，这表明可以对PR进行测试。</li>
<li>对于贡献者，OK-to-test签表示将对其PR运行常规的CI测试。</li>
<li>对于审核人或者成员来说，在PR上贴上“ok-to-test”的标签有更多的意思：
<ul>
<li>需要注意PR是否会浪费我们的CI/CD资源</li>
<li>PR是否值得测试，或者在进行CI / CD流程之前是否需要进行更多更改？</li>
<li>PR是否运行恶意代码滥用我们的资源。</li>
</ul>
</li>
<li>ok-to-test标签，可以减少工作量，并使贡献者更轻松地体验，因为他们可以知道是否有任何失败的测试。</li>
<li>ok-to-test标签还由其它因素决定：
<ul>
<li>PR的大小：
<ul>
<li>If the PR is of <code>size/S</code> or <code>size/M</code> which is just to fix a grammatical error or spelling mistake, the reviewer can trigger the <code>CI/CD</code> without having a second thought.</li>
<li>If the PR is of <code>size/XXL</code> which aims at adding a new feature, a new API endpoint or any new substantial feature. There needs to other conventions &amp; process to be followed regarding the change made. Hence it may have a slight to delay to get labelled with <code>ok-to-test</code>.</li>
</ul>
</li>
<li>如果PR的改动很小，则组织的其他成员也可以添加ok-to-test标记。</li>
<li>如果PR标记为cncf-cla：no，那么最好在标记ok-to-test之前等待。</li>
<li>标有“do-not-merge/hold”或“needs-rebase”标签的PR应当进行适当的更改，然后才能将PR标记为“可以测试”。</li>
<li>错误创建的PR（未进行有意义的代码更改）不应标记为可以测试并关闭。</li>
</ul>
</li>
</ul>
<h5 id="pr生命周期（lifecycle插件）">pr生命周期（lifecycle插件）</h5>
<ul>
<li>
<p><strong>close 指令：关闭issue或者PR</strong></p>
</li>
<li>
<p><strong>reopen指令：开启issue或PR(码云不支持PR重新开启)</strong></p>
</li>
<li>
<p><strong>lifecycle stale指令：标识issue或者PR是陈旧的</strong></p>
<p>例：issue或PR在90天未发生更新则可以使用该指令标识为陈旧的（如不想进入下一阶段 手动移除此标签: <strong>remove-lifecycle stale</strong>）</p>
</li>
<li>
<p><strong>lifecycle rotten指令：标识issue/pr为“臭烂”的</strong></p>
</li>
</ul>
<h5 id="issuepr-生命周期检查任务">issue/PR 生命周期检查任务</h5>
<ul>
<li>
<p>检查是否是issue/PR是否是陈旧的周期任务</p>
<p>根据issue/PR的更新时间、标签以及约定周期检索issue/PR。符合条件在对应的issue/pr添加评论提示 已经在约定周期内没发生更新，并触发 /lifecycle stale指令给到lifecycle组件。</p>
</li>
<li>
<p>检查陈旧状态的PR是否“臭了”周期任务</p>
<p>在上一个任务中打上“lifecycle/stale”标签会更新issue/pr 的更新时间，在此基础上如果PR/issue在约定周期内还未发生更新则在评论区给出提示：在约定周期内未发生更新，并触发/lifecycle rotten指令打上标签，并提示在约定周期内将关闭issue/PR</p>
</li>
<li>
<p>清理issue/pr是“rotten”状态的周期任务</p>
<p>提示在约定周期内未更新PR将关闭 issue/pr 触发/close指令 并提示可以使用reopen指令打开。</p>
</li>
</ul>
<h5 id="pr指定reviewer">PR指定reviewer</h5>
<p>pr指定reviewer 相关插件 <strong>assign</strong>,<strong>blunderbuss</strong></p>
<ul>
<li><strong>assign插件</strong>
<ul>
<li>功能描述：向指定用户指派reviewer或请求review</li>
<li>指令：
<ul>
<li>assign @user 指定user为pr/(issue?待确定)的reviewer</li>
<li>unassign @user 取消指定</li>
<li>cc @user 向user发情review请求（github 会发送邮件）</li>
</ul>
</li>
</ul>
</li>
<li><strong>blunderbuss插件</strong>
<ul>
<li>功能描述：创建新的PR时 会根据PR改动的文件对应的目录下Owners文件指定Reviewer。也可使用指令手动触发。</li>
<li>指令/hook
<ul>
<li>监听PR创建的webhook事件</li>
<li>auto-cc 指令手动触发（触发是否会重新指定人待阅读源码）</li>
</ul>
</li>
</ul>
</li>
</ul>
<h4 id="调研issue参考链接">调研issue<a href="https://github.com/kubernetes/community/blob/master/contributors/guide/issue-triage.md">参考链接</a></h4>
<ol>
<li>review 新创建的问题，对于没有贴分类标签的问题，sig组领导应至少确定一个sig组成员为该问题的第一负责人。</li>
<li>对issue进行分类：贴上分类标签（在组织中预设了分类标签，由分诊人员编辑）</li>
<li>定义优先级：使用label插件 /priority 指令贴标签</li>
<li>将issue分给正确的sig组: 使用label插件 /sig 指令贴标签</li>
<li>跟进issue
<ul>
<li>人工跟进</li>
<li>生命周期任务跟进。</li>
</ul>
</li>
</ol>
<h4 id="调研owners机制">调研owners机制</h4>
<p>使用它们来分配审阅者和批准者角色，每个包含一个独立代码或内容单元的目录也可能包含一个OWNERS文件。该文件适用于目录中的所有内容，包括OWNERS文件本身，同级文件和子目录。</p>
<p>owners 文件配置项说明：</p>
<ul>
<li>approve : 可以在PR使用 /approve指令github用户或别名列表</li>
<li>labels: GitHub标签列表以自动应用于PR</li>
<li>options: 解释owners文件依赖关系的字典
<ul>
<li>no_parent_owners 默认为false，如果为true 排除父owners文件</li>
</ul>
</li>
<li>reviewers: 可以在PR中使用/lgtm指令的github用户或别名列表。</li>
</ul>
<p>OWNERS 文件应定期维护。</p>
<p>建议：</p>
<ul>
<li>增加owners文件数量</li>
<li>为owners文件添加新用户</li>
<li>确保owners文件只包含组织成员</li>
<li>确保OWNERS文件仅包含正在积极贡献或审查其拥有的代码的人员</li>
<li>及时删除不活跃的用户</li>
<li>审核者数量应大于批准人</li>
<li>批准者不在“审阅者”部分中</li>
<li>定期更新的OWNERS文件（每个版本至少更新一次）</li>
</ul>
<p><strong>owners文件由repoowner程序包解析在prow多个组件中使用，owners文件手动维护</strong></p>
<h5 id="解决方案">解决方案</h5>
<ul>
<li>
<p>gitee验证“<strong>刷新</strong>”</p>
<ol>
<li>
<p>PR <strong>编辑操作</strong>和<strong>添加评论</strong>都会刷新更新时间。</p>
</li>
<li>
<p>issue <strong>编辑操作</strong>和<strong>添加评论</strong>都会刷新更新时间。</p>
<p>ps: <strong>编辑操作指 gitte 编辑页面能编辑的所有内容</strong></p>
</li>
</ol>
</li>
<li>
<p>定义</p>
<p>定义生命周期标签来跟踪一个issue/PR的活跃状态，根据生命周期标签所在状态来清理issue：</p>
<ul>
<li>
<p>issue或PR的生命周期标签 “lifecycle/stale (陈旧)、lifecycle/rotten(烂掉的)、lifecycle/frezon(冻结)”</p>
</li>
<li>
<p>issue/PR是否活跃根据<strong>issue/PR更新时间距离当前时间的间隔</strong>:</p>
<ul>
<li>一个issue/PR 从非陈旧状态到陈旧状态 nowTime - updateTime &gt;720h(可配置)</li>
<li>一个issue/PR从陈旧状态到烂掉状态 nowTime - updateTime &gt; 720h(可配置)</li>
<li>结束烂掉状态的issue/PR  nowTime - updateTime &gt; 720h (可配置)</li>
</ul>
</li>
<li>
<p>issue/PR处于冻结状态，则表示issue/PR生命周期将不会随是否活跃而发生改变。</p>
</li>
<li>
<p>非活跃状态的issue到关闭会得到三次自动化提醒(根据提醒内容执行相应指令可以充值状态或reopen)。</p>
<ul>
<li>
<p>当一个issue/PR变成stale将会添加评论提醒：</p>
<pre class=" language-shell"><code class="prism  language-shell">Issue/PR go stale after 90 day of inactivity.
Mark the issue as fresh with `/remove-lifecycle stale`.
Stale issues rotten after an additional 30d of inactivity and eventually close.
If this issue is safe to close now please do so with `/close`.
/lifecycle stale
</code></pre>
</li>
<li>
<p>当一个issue/PR从stale状态变成rotten状态会添加提醒：</p>
<pre class=" language-shell"><code class="prism  language-shell">Stale issues rotten after 30 day of inactivity.
Mark the issue as fresh with `/remove-lifecycle rotten`.
Rotten issues close after an additional 30d of inactivity.

If this issue is safe to close now please do so with `/close`.
/lifecycle rotten
</code></pre>
</li>
<li>
<p>当一个达到清除rotten状态的issue/PR时会添加提醒：</p>
<pre class=" language-shell"><code class="prism  language-shell">#issue 
Rotten issues close after 30d of inactivity.
Reopen the issue with `/reopen`.
/close
#PR 
Rotten PR close after 30d of inactivity.
/close

</code></pre>
</li>
</ul>
</li>
</ul>
</li>
<li>
<p>实现</p>
<ul>
<li>
<p>概要</p>
<p>使用prow支持的periodics job 检查issue/PR的活跃状态，根据检测结果在对应的PR/issue添加评论提醒以及触发修改状态指令。</p>
<p>任务执行程序的配置项：</p>
<pre class=" language-yaml"><code class="prism  language-yaml">  <span class="token key atrule">args</span><span class="token punctuation">:</span>
      <span class="token punctuation">-</span> <span class="token punctuation">-</span><span class="token punctuation">-</span>config=/etc/config/commenter_stale.yaml
      <span class="token punctuation">-</span> <span class="token punctuation">-</span><span class="token punctuation">-</span>token=/etc/token/bot<span class="token punctuation">-</span>github<span class="token punctuation">-</span>token
      <span class="token punctuation">-</span> <span class="token punctuation">-</span><span class="token punctuation">-</span>maxprocess=10

</code></pre>
<p>commenter_stale.yaml:</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">org</span><span class="token punctuation">:</span>
 <span class="token punctuation">-</span> cve<span class="token punctuation">-</span>manage<span class="token punctuation">-</span>test
 <span class="token punctuation">-</span> kndcdc
<span class="token key atrule">repo</span><span class="token punctuation">:</span>
 <span class="token punctuation">-</span> xwz/repo1
 <span class="token punctuation">-</span> xwz/repo2
<span class="token key atrule">exclude_repo</span><span class="token punctuation">:</span>
 <span class="token punctuation">-</span> cve<span class="token punctuation">-</span>manage<span class="token punctuation">-</span>test/config
<span class="token key atrule">uncheck_PR</span><span class="token punctuation">:</span> <span class="token boolean important">false</span>
<span class="token key atrule">uncheck_issue</span><span class="token punctuation">:</span> <span class="token boolean important">false</span>
<span class="token key atrule">checkers</span><span class="token punctuation">:</span>
 <span class="token punctuation">-</span> <span class="token key atrule">label</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> lifecycle/stale
   <span class="token key atrule">exclude_label</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> lifecycle/rotten
   <span class="token key atrule">updated</span><span class="token punctuation">:</span> 720h
   <span class="token key atrule">comment</span><span class="token punctuation">:</span> <span class="token punctuation">|</span><span class="token punctuation">-</span> 
    Issue/PR go stale after 90 day of inactivity.
    Mark the issue as fresh with `/remove<span class="token punctuation">-</span>lifecycle stale`.
    Stale issues rotten after an additional 30d of inactivity and eventually close.
    If this issue is safe to close now please do so with `/close`.
    /lifecycle stale
 
</code></pre>
<ul>
<li>
<p>org 要查询的组织</p>
</li>
<li>
<p>repo 指定要查询的仓库全路径</p>
</li>
<li>
<p>exclude_repo 要排除的仓库全路径</p>
</li>
<li>
<p>checker 任务列表配置</p>
</li>
<li>
<p>label 包含指定标签</p>
</li>
<li>
<p>-label 不包含指定标签</p>
</li>
<li>
<p>uncheck_PR 是否检测PR</p>
</li>
<li>
<p>uncheck_issue 是否检测issue</p>
</li>
<li>
<p>updated: 满足检查项的未活动时间</p>
</li>
<li>
<p>comment: 添加评论的内容</p>
</li>
</ul>
</li>
<li>
<p>需调用的giteeAPI(带授权token 每天限制访问10000次)</p>
<ul>
<li>根据组织名获取所有仓库：</li>
<li>根据仓库获取所有open状态的PR：</li>
<li>根据仓库获取所有open状态的issue:</li>
<li>给ISSUE添加评论:</li>
<li>给PR添加评论：<br>
<strong>建议：增加访问的次数，使用与其他业务不同的TOKEN（减少对核心功能的影响）</strong></li>
</ul>
</li>
<li>
<p>程序流程图</p>
<p><img src="http://www.plantuml.com/plantuml/png/SoWkIImgAStDuNgoO_VpNVkVh-XMqBLJW7Ei55wkwdcnll5bQ-_plUlG5EsU_7JNvATBMfvFQ8E8Fjkum4hXQSVifxjt6JxPiGfT-UwdNGlnA2TaEaoj7qWjiZHX9tKjUDgwr8-Gap3jG4YZQy9h1wO_dgzR-B9nifN2YxxjJ_kdGSIUJTa7r84ipuNCKR3HnHe8A6RW3GTqZsrxsR0WEICrEHlY1Hnw8P8m1nUKbYYWwC4C4njTD3tfgRZru0_yPvtBNopiGTRha9gN0aoU0000" alt="pic"></p>
</li>
</ul>
</li>
</ul>

