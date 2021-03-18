---


---

<h2 id="travis-ci-体验">travis CI 体验</h2>
<h4 id="快速使用（for-github）">快速使用（for github）</h4>
<ol>
<li>
<p>使用github账号在travis-ci.com进行注册</p>
</li>
<li>
<p>接受TravisCI授权。</p>
</li>
<li>
<p>单击Travis仪表板右上方的个人资料图片，单击“设置”，然后单击绿色的“激活”按钮，然后选择要用于Travis CI的存储库。</p>
</li>
<li>
<p>在你的代码仓库中添加 .travis.yml文件以告知travis要做什么，以下示例指定了应使用Ruby 2.2和最新版本的JRuby构建的Ruby项目：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token comment"># .travis.yml</span>
<span class="token key atrule">language</span><span class="token punctuation">:</span> ruby
<span class="token key atrule">rvm</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> <span class="token number">2.2</span>
  <span class="token punctuation">-</span> jruby
</code></pre>
</li>
<li>
<p>添加该文件到git,提交并push到你的远程仓库触发travis构建</p>
</li>
<li>
<p>通过访问Travis CI并选择您的存储库，检查构建状态页面，以根据构建命令的返回状态查看您的构建是否通过或失败。</p>
</li>
</ol>
<h4 id="使用docker构建">使用docker构建</h4>
<p>此示例存储库运行从同一映像构建的两个Docker容器：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token comment">#.travis.yml</span>
<span class="token key atrule">language</span><span class="token punctuation">:</span> ruby

<span class="token key atrule">services</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> docker

<span class="token key atrule">before_install</span><span class="token punctuation">:</span>
<span class="token punctuation">-</span> docker pull carlad/sinatra
<span class="token punctuation">-</span> docker run <span class="token punctuation">-</span>d <span class="token punctuation">-</span>p 127.0.0.1<span class="token punctuation">:</span>80<span class="token punctuation">:</span>4567 carlad/sinatra /bin/sh <span class="token punctuation">-</span>c "cd /root/sinatra; bundle exec foreman start;"
<span class="token punctuation">-</span> docker ps <span class="token punctuation">-</span>a
<span class="token punctuation">-</span> docker run carlad/sinatra /bin/sh <span class="token punctuation">-</span>c "cd /root/sinatra; bundle exec rake test"

<span class="token key atrule">script</span><span class="token punctuation">:</span>
<span class="token punctuation">-</span> bundle exec rake test
</code></pre>
<h5 id="通过dockerfile构建docker-image">通过dockerfile构建docker image</h5>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token comment">#.travis.yml</span>
<span class="token key atrule">language</span><span class="token punctuation">:</span> ruby
<span class="token key atrule">services</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> docker

<span class="token key atrule">before_install</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> docker build <span class="token punctuation">-</span>t carlad/sinatra .
  <span class="token punctuation">-</span> docker run <span class="token punctuation">-</span>d <span class="token punctuation">-</span>p 127.0.0.1<span class="token punctuation">:</span>80<span class="token punctuation">:</span>4567 carlad/sinatra /bin/sh <span class="token punctuation">-</span>c "cd /root/sinatra; bundle exec foreman start;"
  <span class="token punctuation">-</span> docker ps <span class="token punctuation">-</span>a
  <span class="token punctuation">-</span> docker run carlad/sinatra /bin/sh <span class="token punctuation">-</span>c "cd /root/sinatra; bundle exec rake test"

<span class="token key atrule">script</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> bundle exec rake test
</code></pre>
<h5 id="推送docker-image-到镜像仓">推送docker image 到镜像仓</h5>
<p>推送到镜像仓首先要通过 docker login 进行身份验证，用于登录的电子邮件，用户名和密码应存储在存储库设置环境变量中，该变量可以通过存储库设置网页设置，也可以通过Travis CLI在本地设置，例如：</p>
<pre class=" language-shell"><code class="prism  language-shell">travis env set DOCKER_USERNAME myusername
travis env set DOCKER_PASSWORD secretsecret
</code></pre>
<p>确保您已经使用 travis gem 加密环境变量。</p>
<p>要将存储库的特定分支推送到远程注册表，请使用.travis.yml的定制部署部分：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token comment">#.travis.yaml</span>
<span class="token key atrule">deploy</span><span class="token punctuation">:</span>
  <span class="token key atrule">provider</span><span class="token punctuation">:</span> script
  <span class="token key atrule">script</span><span class="token punctuation">:</span> bash docker_push
  <span class="token key atrule">on</span><span class="token punctuation">:</span>
    <span class="token key atrule">branch</span><span class="token punctuation">:</span> master
</code></pre>
<p>其中docker_push是存储库中的脚本：</p>
<pre class=" language-shell"><code class="prism  language-shell">#!/bin/bash
echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
docker push USER/REPO
</code></pre>
<p>当推送到私有注册表时，请确保在docker login命令中指定主机名，例如：</p>
<pre class=" language-shell"><code class="prism  language-shell">echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin registry.example.com
</code></pre>
<h4 id="构建pullrequest">构建PullRequest</h4>
<p>PR构建是Travis CI的重要组成部分。每当在GitHub上打开PR时，Travis CI都会构建该请求并更新拉取请求页面上的状态图标。</p>
<p>travis ci 收到PR通知并运行构建。在构建期间会更新PR状态为以下状态之一：</p>
<ul>
<li>构建正在运行的警告</li>
<li>构建失败PR不应该被合并的通知</li>
<li>构建成功PR可以被合入</li>
</ul>
<p>首次打开Travis CI时，只要将提交添加到该PR中，它就会构建一个PR。我们构建源分支和上游分支之间的合并，而不是构建已被推送到PR所来自的分支的提交。</p>
<p>要仅在推送事件上构建而不在拉取请求上构建，请在存储库设置中禁用“在PR上构建”。</p>
<p>要仅在特定分支上构建PR可以使用 branchs: only: 键来指定</p>
<h5 id="pr-构建的安全限制">PR 构建的安全限制</h5>
<p>因一个从上游仓库fork的仓库提交的PR可以操纵暴露环境变量，而上游仓库维护者无法抵御这种攻击。travis CI 设定为外部PR(fork 仓提交的PR)无法访问环境变量和加密数据。即使这些已在派生源项目中定义。</p>
<p>要解决此问题，请将这些测试仅限制在环境变量可用的情况下：</p>
<pre class=" language-shell"><code class="prism  language-shell">script:
   - 'if [ "$TRAVIS_PULL_REQUEST" != "false" ]; then bash ./travis/run_on_pull_requests; fi'
   - 'if [ "$TRAVIS_PULL_REQUEST" = "false" ]; then bash ./travis/run_on_non_pull_requests; fi'
</code></pre>
<h4 id="cron-job">Cron Job</h4>
<p>Travis CI cron作业的工作方式与cron实用程序相似，它们以固定的预定时间间隔运行构建，而与是否将任何提交推送到存储库无关。Cron作业始终获取特定分支上的最新提交，并在该状态下构建项目。Cron作业可以每天，每周或每月运行，这实际上意味着在选定的时间跨度后最多一个小时，您不能将其设置为在特定时间运行。</p>
<p>请通过Travis CI网页上的“ Cron Jobs”设置选项卡配置cron作业。</p>
<h4 id="job-的生命周期">Job 的生命周期</h4>
<p>Travis CI为每种编程语言提供了默认的构建环境和默认的阶段集。将使用您的工作的构建环境创建一个虚拟机，将您的存储库克隆到其中，安装可选的附加组件，然后运行构建阶段。</p>
<p>每个作业都是一个阶段序列。主要阶段是：</p>
<ol>
<li>install 安装所需的任何依赖项</li>
<li>script 运行构建脚本</li>
</ol>
<p>Travis CI可以在以下阶段中运行自定义命令：</p>
<ol>
<li>befor_install 在安装阶段之前</li>
<li>befor_script 在运行脚本之前</li>
<li>after_script 在运行脚本之后</li>
<li>after_success 当构建成功时（比如： 构建文档）结果在TRAVIS_TEST_RESULT环境变量中</li>
<li>after_failure 当构建失败时 （比如：更新log文件） 结果在TRAVIS_TEST_RESULT环境变量中</li>
</ol>
<p>有三个可选的部署阶段。</p>
<p>其完整的构建阶段（生命周期）：</p>
<ol>
<li>可选安装 apt addons</li>
<li>可选安装 cache components</li>
<li>befor install</li>
<li>install</li>
<li>before_script</li>
<li>script</li>
<li>可选 befor_cache（当且仅当缓存有效时）</li>
<li>after_success or after_failure</li>
<li>可选 before_depoly</li>
<li>可选depoly</li>
<li>可选after_depoly</li>
<li>after_script</li>
</ol>
<p>NOTE: 一个构建可以包含许多工作。</p>
<h4 id="构建矩阵">构建矩阵</h4>
<p>一个构建矩阵由几个并行运行的多个作业组成。</p>
<p>在许多情况下这可能很有用，但是使用构建矩阵的两个主要原因是：</p>
<ul>
<li>减少整体的构建时间</li>
<li>针对不同版本的运行时或依赖项运行测试</li>
</ul>
<p>此页面上的示例着重于后一种用例。</p>
<p>有两种方法可以在.travis.yml文件中定义矩阵，这两个功能可以组合使用：</p>
<ul>
<li>使用矩阵扩展功能</li>
<li>列出单个作业配置</li>
</ul>
<h5 id="扩展矩阵">扩展矩阵</h5>
<p>某些键定义为采用值数组的矩阵扩展键，从而为每个值创建一个额外的作业。如果提供了几个矩阵扩展键，则它将乘以创建的作业数量。</p>
<p>例如，以下配置生成一个构建矩阵，该矩阵扩展为8个单独的（2 * 2 * 2）作业，将来自三个矩阵扩展键rvm，gemfile和env的每个值组合在一起。</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">rvm</span><span class="token punctuation">:</span>
<span class="token punctuation">-</span> <span class="token number">2.5</span>
<span class="token punctuation">-</span> <span class="token number">2.2</span>
<span class="token key atrule">gemfile</span><span class="token punctuation">:</span>
<span class="token punctuation">-</span> gemfiles/Gemfile.rails<span class="token punctuation">-</span>3.2.x
<span class="token punctuation">-</span> gemfiles/Gemfile.rails<span class="token punctuation">-</span>3.0.x
<span class="token key atrule">env</span><span class="token punctuation">:</span>
<span class="token punctuation">-</span> ISOLATED=true
<span class="token punctuation">-</span> ISOLATED=false
</code></pre>
<h5 id="列出单个作业配置">列出单个作业配置</h5>
<p>另外，可以通过将条目添加到关键作业.include中来指定作业。例如，如果上面矩阵扩展的所有组合都不是全部相关，则可以像下面这样单独指定作业：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">jobs</span><span class="token punctuation">:</span>
  <span class="token key atrule">include</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> <span class="token key atrule">rvm</span><span class="token punctuation">:</span> <span class="token number">2.5</span>
    <span class="token key atrule">gemfile</span><span class="token punctuation">:</span> gemfiles/Gemfile.rails<span class="token punctuation">-</span>3.2.x
    <span class="token key atrule">env</span><span class="token punctuation">:</span> ISOLATED=false
  <span class="token punctuation">-</span> <span class="token key atrule">rvm</span><span class="token punctuation">:</span> <span class="token number">2.2</span>
    <span class="token key atrule">gemfile</span><span class="token punctuation">:</span> gemfiles/Gemfile.rails<span class="token punctuation">-</span>3.0.x
    <span class="token key atrule">env</span><span class="token punctuation">:</span> ISOLATED=true
</code></pre>
<p><em>NOTE: 对于私有和公共存储库，当前构建矩阵的最大限制为200个JOBS。如果您采用开源计划，请记住Travis CI向社区免费提供此服务。因此，请仅指定您实际需要的矩阵。</em></p>
<h5 id="排除job">排除JOB</h5>
<p>构建矩阵扩展有时会产生不需要的组合。在这种情况下，使用键jobs.exclude排除某些组合会比较方便，而不是单独列出所有作业。例如，这将从构建矩阵中排除两个作业：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">jobs</span><span class="token punctuation">:</span>
  <span class="token key atrule">exclude</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> <span class="token key atrule">rvm</span><span class="token punctuation">:</span> 1.9.3
    <span class="token key atrule">gemfile</span><span class="token punctuation">:</span> gemfiles/Gemfile.rails<span class="token punctuation">-</span>2.3.x
    <span class="token key atrule">env</span><span class="token punctuation">:</span> ISOLATED=true
  <span class="token punctuation">-</span> <span class="token key atrule">rvm</span><span class="token punctuation">:</span> jruby
    <span class="token key atrule">gemfile</span><span class="token punctuation">:</span> gemfiles/Gemfile.rails<span class="token punctuation">-</span>2.3.x
    <span class="token key atrule">env</span><span class="token punctuation">:</span> ISOLATED=true
</code></pre>
<p>如果要从构建矩阵中排除的作业共享相同的矩阵参数，则可以仅指定那些参数，并忽略不同的部分。假设您有：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">language</span><span class="token punctuation">:</span> ruby
<span class="token key atrule">rvm</span><span class="token punctuation">:</span>
<span class="token punctuation">-</span> 1.9.3
<span class="token punctuation">-</span> 2.0.0
<span class="token punctuation">-</span> 2.1.0
<span class="token key atrule">env</span><span class="token punctuation">:</span>
<span class="token punctuation">-</span> DB=mongodb
<span class="token punctuation">-</span> DB=redis
<span class="token punctuation">-</span> DB=mysql
<span class="token key atrule">gemfile</span><span class="token punctuation">:</span>
<span class="token punctuation">-</span> Gemfile
<span class="token punctuation">-</span> gemfiles/rails4.gemfile
<span class="token punctuation">-</span> gemfiles/rails31.gemfile
<span class="token punctuation">-</span> gemfiles/rails32.gemfile
</code></pre>
<p>这产生了一个3×3×4的构建矩阵。要排除所有具有rvm值2.0.0和gemfile值Gemfile的作业，您可以编写：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">jobs</span><span class="token punctuation">:</span>
  <span class="token key atrule">exclude</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> <span class="token key atrule">rvm</span><span class="token punctuation">:</span> 2.0.0
    <span class="token key atrule">gemfile</span><span class="token punctuation">:</span> Gemfile
</code></pre>
<p>等价于：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">jobs</span><span class="token punctuation">:</span>
  <span class="token key atrule">exclude</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> <span class="token key atrule">rvm</span><span class="token punctuation">:</span> 2.0.0
    <span class="token key atrule">gemfile</span><span class="token punctuation">:</span> Gemfile
    <span class="token key atrule">env</span><span class="token punctuation">:</span> DB=mongodb
  <span class="token punctuation">-</span> <span class="token key atrule">rvm</span><span class="token punctuation">:</span> 2.0.0
    <span class="token key atrule">gemfile</span><span class="token punctuation">:</span> Gemfile
    <span class="token key atrule">env</span><span class="token punctuation">:</span> DB=redis
  <span class="token punctuation">-</span> <span class="token key atrule">rvm</span><span class="token punctuation">:</span> 2.0.0
    <span class="token key atrule">gemfile</span><span class="token punctuation">:</span> Gemfile
    <span class="token key atrule">env</span><span class="token punctuation">:</span> DB=mysql
</code></pre>
<h4 id="构建阶段">构建阶段</h4>
<p>构建阶段是一种对作业进行分组的方法，可以在每个阶段并行运行作业，但是依次运行一个阶段，并且只有在上一个阶段的所有作业都成功通过后才能继续进行。</p>
<p>这是在.travis.yml文件中为此设置构建配置的方式：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">jobs</span><span class="token punctuation">:</span>
  <span class="token key atrule">include</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> <span class="token key atrule">stage</span><span class="token punctuation">:</span> test
      <span class="token key atrule">script</span><span class="token punctuation">:</span> ./test 1
    <span class="token punctuation">-</span> <span class="token comment"># stage name not required, will continue to use `test`</span>
      <span class="token key atrule">script</span><span class="token punctuation">:</span> ./test 2
    <span class="token punctuation">-</span> <span class="token key atrule">stage</span><span class="token punctuation">:</span> deploy
      <span class="token key atrule">script</span><span class="token punctuation">:</span> ./deploy
</code></pre>
<p>您还可以在构建阶段中命名特定作业。我们建议使用唯一的工作名称，但不要强制使用（尽管将来可能会更改）。可以为Jobs.include节中定义的作业指定名称属性，如下所示：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">jobs</span><span class="token punctuation">:</span>
  <span class="token key atrule">include</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> <span class="token key atrule">stage</span><span class="token punctuation">:</span> "Tests"                <span class="token comment"># naming the Tests stage</span>
      <span class="token key atrule">name</span><span class="token punctuation">:</span> "Unit Tests"            <span class="token comment"># names the first Tests stage job</span>
      <span class="token key atrule">script</span><span class="token punctuation">:</span> ./unit<span class="token punctuation">-</span>tests
    <span class="token punctuation">-</span> <span class="token key atrule">script</span><span class="token punctuation">:</span> ./integration<span class="token punctuation">-</span>tests
      <span class="token key atrule">name</span><span class="token punctuation">:</span> "Integration Tests"     <span class="token comment"># names the second Tests stage job</span>
    <span class="token punctuation">-</span> <span class="token key atrule">stage</span><span class="token punctuation">:</span> deploy
      <span class="token key atrule">name</span><span class="token punctuation">:</span> <span class="token string">"Deploy to GCP"</span>
      <span class="token key atrule">script</span><span class="token punctuation">:</span> ./deploy
</code></pre>
<h4 id="有条件-构建、stage以及作业">有条件 构建、Stage以及作业</h4>
<h5 id="条件构建">条件构建</h5>
<p>您可以将Travis CI配置为仅在满足某些条件时运行构建</p>
<p>例如，这允许构建仅在master分支上运行：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token comment"># require the branch name to be master (note for PRs this is the base branch name)</span>
<span class="token key atrule">if</span><span class="token punctuation">:</span> branch = master
</code></pre>
<h5 id="conditional-stage">conditional Stage</h5>
<p>您可以将Travis CI配置为仅包含满足特定条件的阶段。不符合给定条件的阶段将被静默跳过。例如，这允许部署阶段仅在master分支上运行：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">stages</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> <span class="token key atrule">name</span><span class="token punctuation">:</span> deploy
    <span class="token comment"># require the branch name to be master (note for PRs this is the base branch name)</span>
    <span class="token key atrule">if</span><span class="token punctuation">:</span> branch = master
</code></pre>
<h5 id="conditional-jobs">Conditional Jobs</h5>
<p>例如，这仅包括列出的作业以建立在master分支上：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">jobs</span><span class="token punctuation">:</span>
  <span class="token key atrule">include</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> <span class="token comment"># require the branch name to be master (note for PRs this is the base branch name)</span>
      <span class="token key atrule">if</span><span class="token punctuation">:</span> branch = master
      <span class="token key atrule">env</span><span class="token punctuation">:</span> FOO=foo
</code></pre>
<h4 id="安装依赖">安装依赖</h4>
<h5 id="在基础设施上安装软件包">在基础设施上安装软件包</h5>
<p>在Ubuntu上使用apt 安装软件包：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">before_install</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> sudo apt<span class="token punctuation">-</span>get <span class="token punctuation">-</span>y install libxml2<span class="token punctuation">-</span>dev
</code></pre>
<p>默认情况下，apt-get更新不会自动运行。如果要在每个版本上自动更新apt-get update，则有两种方法可以执行此操作。首先是通过在before_install步骤中显式运行apt-get update：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">before_install</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> sudo apt<span class="token punctuation">-</span>get update
  <span class="token punctuation">-</span> sudo apt<span class="token punctuation">-</span>get <span class="token punctuation">-</span>y install libxml2<span class="token punctuation">-</span>dev
</code></pre>
<p>第二种方法是使用APT插件：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">before_install</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> sudo apt<span class="token punctuation">-</span>get <span class="token punctuation">-</span>y install libxml2<span class="token punctuation">-</span>dev
<span class="token key atrule">addons</span><span class="token punctuation">:</span>
  <span class="token key atrule">apt</span><span class="token punctuation">:</span>
    <span class="token key atrule">update</span><span class="token punctuation">:</span> <span class="token boolean important">true</span>
</code></pre>
<p><em>NOTE:不要在构建中运行apt-get升级，因为它会下载多达500MB的软件包，并且会大大延长构建时间。此外，某些软件包可能无法更新，这将导致构建失败。</em></p>
<h5 id="从自定义apt存储库安装软件包">从自定义APT存储库安装软件包</h5>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">before_install</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> sudo add<span class="token punctuation">-</span>apt<span class="token punctuation">-</span>repository <span class="token punctuation">-</span>y ppa<span class="token punctuation">:</span>ubuntu<span class="token punctuation">-</span>toolchain<span class="token punctuation">-</span>r/test
  <span class="token punctuation">-</span> sudo apt<span class="token punctuation">-</span>get <span class="token punctuation">-</span>q update
  <span class="token punctuation">-</span> sudo apt<span class="token punctuation">-</span>get <span class="token punctuation">-</span>y install gcc<span class="token punctuation">-</span><span class="token number">4.8</span>
</code></pre>
<p>对于不在Launchpad上托管的存储库，您还需要添加一个GnuPG密钥。如果您是以这种方式安装软件包，请确保下载适合您环境的正确版本。</p>
<p>本示例将适用于Ubuntu 12.04的Varnish 3.0的APT存储库添加到APT源的本地可用列表中，然后安装varnish软件包。</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">before_script</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> curl http<span class="token punctuation">:</span>//repo.varnish<span class="token punctuation">-</span>cache.org/debian/GPG<span class="token punctuation">-</span>key.txt <span class="token punctuation">|</span> sudo apt<span class="token punctuation">-</span>key add <span class="token punctuation">-</span>
  <span class="token punctuation">-</span> echo "deb http<span class="token punctuation">:</span>//repo.varnish<span class="token punctuation">-</span>cache.org/ubuntu/ precise varnish<span class="token punctuation">-</span>3.0" <span class="token punctuation">|</span> sudo tee <span class="token punctuation">-</span>a /etc/apt/sources.list
  <span class="token punctuation">-</span> sudo apt<span class="token punctuation">-</span>get <span class="token punctuation">-</span>qq update
  <span class="token punctuation">-</span> sudo apt<span class="token punctuation">-</span>get <span class="token punctuation">-</span>y install varnish
</code></pre>
<h5 id="在没有apt存储库的情况下安装软件包">在没有APT存储库的情况下安装软件包</h5>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">before_install</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> wget http<span class="token punctuation">:</span>//pngquant.org/pngquant_1.7.1<span class="token punctuation">-</span>1_i386.deb
  <span class="token punctuation">-</span> sudo dpkg <span class="token punctuation">-</span>i pngquant_1.7.1<span class="token punctuation">-</span>1_i386.deb
</code></pre>
<h5 id="使用apt插件安装软件包">使用APT插件安装软件包</h5>
<p>您也可以使用APT插件安装软件包和源，而无需在before_install脚本中运行apt-get命令。</p>
<p>要添加APT源，可以使用以下三种类型之一：</p>
<ol>
<li>aliases defined in <a href="https://github.com/travis-ci/apt-source-safelist">source safelist</a></li>
<li><code>sourceline</code> key-value pairs which will be added to <code>/etc/apt/sources.list</code></li>
<li>when APT sources require GPG keys, you can specify this with <code>key_url</code> pairs in addition to <code>sourceline</code>.</li>
</ol>
<p>以下代码片段显示了这三种类型的APT来源</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">addons</span><span class="token punctuation">:</span>
  <span class="token key atrule">apt</span><span class="token punctuation">:</span>
    <span class="token key atrule">sources</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> deadsnakes
    <span class="token punctuation">-</span> <span class="token key atrule">sourceline</span><span class="token punctuation">:</span> <span class="token string">'ppa:ubuntu-toolchain-r/test'</span>
    <span class="token punctuation">-</span> <span class="token key atrule">sourceline</span><span class="token punctuation">:</span> <span class="token string">'deb https://packagecloud.io/chef/stable/ubuntu/precise main'</span>
      <span class="token key atrule">key_url</span><span class="token punctuation">:</span> <span class="token string">'https://packagecloud.io/gpg.key'</span>
</code></pre>
<p>添加APT软件包</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">addons</span><span class="token punctuation">:</span>
  <span class="token key atrule">apt</span><span class="token punctuation">:</span>
    <span class="token key atrule">packages</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> cmake
    <span class="token punctuation">-</span> time
</code></pre>
<p><em>Note: When using APT sources and packages together, you need to make sure they are under the same key space in the YAML file. e.g.</em></p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">addons</span><span class="token punctuation">:</span>
  <span class="token key atrule">apt</span><span class="token punctuation">:</span>
    <span class="token key atrule">sources</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> ubuntu<span class="token punctuation">-</span>toolchain<span class="token punctuation">-</span>r<span class="token punctuation">-</span>test
    <span class="token key atrule">packages</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> gcc<span class="token punctuation">-</span><span class="token number">4.8</span>
    <span class="token punctuation">-</span> g++<span class="token punctuation">-</span><span class="token number">4.8</span>
</code></pre>
<p>略…</p>
<h4 id="缓存依赖项和目录">缓存依赖项和目录</h4>
<p>Travis CI可以缓存不经常更改的内容，以加快构建过程。要使用缓存功能，请在存储库设置中将“构建推送的分支”设置为“开”。</p>
<h5 id="enabling-bundler-caching-点击查看更多">Enabling Bundler caching <a href="https://docs.travis-ci.com/user/caching/#enabling-bundler-caching">点击查看更多</a></h5>
<p>To enable Bundler caching in your <code>.travis.yml</code>:</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">language</span><span class="token punctuation">:</span> ruby
<span class="token key atrule">cache</span><span class="token punctuation">:</span> bundler
</code></pre>
<h4 id="添加ssh到已知主机">添加ssh到已知主机</h4>
<p><a href="https://docs.travis-ci.com/user/ssh-known-hosts/">点击查看更多</a></p>
<h4 id="编程语言">编程语言</h4>
<p><a href="https://docs.travis-ci.com/user/languages/">点击查看更多</a></p>
<h4 id="部署上传">部署上传</h4>
<p><a href="https://docs.travis-ci.com/user/deployment/">点击查看更多</a></p>
<h4 id="ci构建环境">CI构建环境</h4>
<h5 id="虚拟构建环境">虚拟构建环境</h5>
<p>每个构建都在以下虚拟环境之一中运行。</p>
<h6 id="linux">linux</h6>
<p>Travis CI支持两种用于Linux构建的虚拟化类型：“完整VM”和“ LXD”。最重要的是，Linux构建可以在多种CPU架构上运行。</p>
<ul>
<li>
<p>full VM</p>
<p>This is sudo enabled, full virtual machine per build, that runs Linux</p>
</li>
<li>
<p>LXD 容器</p>
<p>这是启用了sudo的LXD容器构建环境，您可以在容器世界中尽可能接近虚拟机。 Linux环境在无特权的LXD容器中运行。</p>
</li>
</ul>
<h6 id="macos">macOS</h6>
<p>适用于Objective-C和其他macOS特定项目的macOS环境</p>
<h6 id="windows">windows</h6>
<p>运行Windows Server版本1803的Windows环境。</p>
<h6 id="虚拟化环境与操作系统-点击查看更多">虚拟化环境与操作系统 <a href="https://docs.travis-ci.com/user/reference/overview/#virtualisation-environment-vs-operating-system">点击查看更多</a></h6>
<ul>
<li>
<p>使用 ubutun Xenial (可供选择的版本请点击查看更多)</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">dist</span><span class="token punctuation">:</span> xenial
</code></pre>
</li>
<li>
<p>使用macos</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">os</span><span class="token punctuation">:</span> osx
</code></pre>
</li>
<li>
<p>使用windows</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">os</span><span class="token punctuation">:</span> windows
</code></pre>
</li>
<li>
<p>多操作系统使用（支持 linux和macos）</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">os</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> linux
  <span class="token punctuation">-</span> osx
</code></pre>
<p>要忽略一个操作系统上作业的结果，请将以下内容添加到您的.travis.yml中：</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">jobs</span><span class="token punctuation">:</span>
  <span class="token key atrule">allow_failures</span><span class="token punctuation">:</span>
    <span class="token punctuation">-</span> <span class="token key atrule">os</span><span class="token punctuation">:</span> osx
</code></pre>
</li>
<li>
<p>在不同的CPU架构上构建</p>
<pre class=" language-yaml"><code class="prism  language-yaml"><span class="token key atrule">arch</span><span class="token punctuation">:</span>
  <span class="token punctuation">-</span> amd64
  <span class="token punctuation">-</span> ppc64le
  <span class="token punctuation">-</span> s390x
  <span class="token punctuation">-</span> arm64  <span class="token comment"># please note arm64-graviton2 requires explicit virt: [lxd|vm] tag so it's recommended for jobs.include, see below</span>
<span class="token key atrule">os</span><span class="token punctuation">:</span> linux  <span class="token comment"># different CPU architectures are only supported on Linux</span>
</code></pre>
</li>
</ul>
<h4 id="travis-ci构建配置参考">Travis CI构建配置参考</h4>
<p><a href="https://config.travis-ci.com/">参考链接</a></p>

