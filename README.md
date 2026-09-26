<h3 align="center">基于改动意图的 git 提交与其消费</h3>

<p align="center">
让你的 git 提交历史变成对 AI Agent 起决定性帮助的资产
</p>

<p align="center">
  <span><strong>简体中文</strong></span> ·
  <a href="README.en.md"><strong>English</strong></a>
</p>

<br/>

这个项目只包含 2 个符合 [Agent Skills 规范](https://agentskills.io/specification) 的 skill，不包含运行时脚本，也不要求特定技术栈、MCP 服务。  

其中一个 skill 用于指导你的 AI Agent 创建精心设计过的 git commit，这些 commit 被称之为 `intent-commit`，其会逐渐在你的项目中形成一个知识库。而另一个 skill 则指导你的 AI Agent 在合适的时机消费它。  

快捷安装这两个 skill 的命令，以及使用方法可见[文末的章节](#安装和使用)。下面会阐述这两个 skill 的意义、设计思路以及工作机制。  

## 为什么要做这个项目

高效、精准的上下文对 AI Agent 顺利完成任务至关重要。  

AI Agent 在执行任务的过程中，可能会进行若干次 Agentic Search 来收集它需要的信息。但要触及真正对解决问题有帮助的那个源码文件里的段落，通常要经过多轮，而这会让宝贵的上下文中充满无关的内容。  即便如此，AI 也只能看到源码表面的情况，很难像人类工程师那样洞察它背后的设计意图。  

正因如此，有人会主动为 AI 维护“项目文档 / 知识库”，亦或者是做“沉淀经验”之类的事，方便 AI 能在遇到问题的时候直接复用已有的经验和知识背景，而不是在源码的海洋里探索。  

> [!NOTE]
> Codex 和 Claude 在今年 2 月也陆续上线了和 Memory 有关的 feature。  
> 例如 Codex 会在 `~/.codex/memories/` 下，存储包括摘要、持久条目、近期输入，以及来自先前聊天的支持性证据。见 [Codex 官方 Memories 文档](https://learn.chatgpt.com/docs/customization/memories?surface=cli#cli-local-memory-storage)  
> 
> 另一个类似的工具是 [Entire](https://entire.io/)，它会在每次提交的时候保留 AI session 的完整记录，在需要消费的时候可以进行语义化检索。  

而我的思路很简单：**直接利用 git 提交信息本身**。  

我在自己维护的企业级项目里实践了这种方式很久，经常观察到它在关键时刻给 AI 提供了至关重要的信息。除此之外，这种方式相比于其他几种甚至还有些额外的优势：  

1. 无需安装额外的 CLI / MCP / Hook 工具，所以对任何 AI Harness 都适用。  
2. 零配置、简单易用也易于理解，无需搭建复杂的知识沉淀工作流也能获得一个高质量的项目知识库。    
3. 对你和你团队的日常工作流几乎没什么侵入性。并且你没有安装此 skill 的同事也能受益，因为提交信息本身就在仓库里，任何人和任何 Agent 都能读到。  

## 是什么，会做什么

### 1. [intent-commit skill](skills/intent-commit/SKILL.md)，生产者  

你和 AI 的 1 个会话往往是围绕一个主题的（把所有互不相关的任务全部塞进同一个 ai 会话本身就不被建议）。当这个会话中的任务完成的时候，你需要手动引用一次 intent-commit skill。  

AI 会把**仅与本次会话相关的改动**，拆分成 1 个或多个符合 [Conventional Commits 规范](https://www.conventionalcommits.org/en/v1.0.0/) 的 git commit。  每一个 commit 的提交信息都是被精心设计过的，它往往采用这种格式：  

```text
<type>(<scope>): <目的、问题修复或行为变化>

<必要的背景与动机；没有额外信息时省略>

实现与决策：
- <关键方案、机制或有依据的取舍>

验证：
- <实际执行的检查、结果与适用范围>

边界与后续：
- <重要约束、未覆盖范围、风险或必要后续工作>

<可选的 Conventional Commits footer>
```

我对于这其中每一部分的设计思考如下：  

#### 记录意图

相比于 vibe coding 时代之前人们往往会在提交信息中写“这次做了什么”，intent-commit 的提交信息主要聚焦于“**这次想做什么**”，也就是改动意图。  

之所以这样设计，是因为我发现在 Vibe Coding 时代，一个程序的真正“源码”不是编程语言，而是指导 AI 一步步构建项目过程中的 prompt，以及 prompt 中包含的架构决策、技术选项和 trade-off 过程。  

这些信息不仅对未来执行任务的 AI 有用，对想读 commit 历史的人类也有用。我发现在这个时代，我看程序源码的次数已经越来越少，反而更希望看“当时为什么要这么做”，“当时是以什么思路和方向指导这次改动的，有过什么关键决策”。  

对于 AI 来说，View File History 的时候，能看到当时改这个文件时候背后的意图和背景，也能让其更像架构师一样帮你维护项目和编写代码，而不是总是局限于局部最优解来制造无数次 patch。  

#### 记录验证

在使用 AI 构建和维护的项目里，明确的验证和测试变得越来越重要与常见。  

你的 Agent 在会话中很可能已经进行过各种测试验证了，让 AI 顺手把它记录下对未来 AI 排障非常有价值，因为它可以判断：

> 当年这个 commit **声称保证了什么行为**？

甚至以后测试坏了，它可以迅速发现：

> “当年 specifically 验证过这个路径，现在这个路径又坏了。”  

#### 记录本次影响与后续建议

在这个部分，AI 会记录本次的改动可能对系统造成什么影响，实现过程中的关键 trade-off，以及其他值得未来维护者或 AI Agent 注意的内容。  

这在未来的 AI Agent 排查回归、解释特殊逻辑、修改既有行为或判断兼容约束时候会起关键作用。  

### 2. [intent-history](skills/intent-history/SKILL.md) skill，消费者

这个 skill 专门用来指导 AI 消费 intent-commit skill 产出的意图式提交信息。  

它会让 AI 在合适的时机用合适的方式查阅 git 历史提交信息，力求最大化 intent-commit 给 AI 解决任务带来的收益。  

消费的形式主要分为两部分：  

1. 语义化关键词搜索

AI 会使用关键词检索既往的 git commit msg 来帮助定位关键源码并同时获取意图式提交信息。这个效果有点类似于 [entire search 命令](https://docs.entire.io/cli-reference/search#search)。  

2. 对文件 / 文件夹进行历史追溯

这通常发生在 AI 已经定位到了关键源码文件。这时候获取与该文件有关的 intent-commit 提交信息，来了解相关背景。   

## 安装和使用

> [!TIP]
> 不推荐一开始就把两个 skill 都装上，因为你之前的项目 git commit 历史可能并不符合 intent-history skill 的需要。  
> 
> 推荐先安装 intent-commit skill，并使用它积累足够的 intent-commits 后，再补充安装 intent-history skill。  

### 安装和使用 intent-commit skill

可以用 [skills CLI](https://github.com/vercel-labs/skills) 一条命令装到几乎所有主流 AI Agent：

```bash
# 安装到当前项目（会写入 ./.claude/skills/ 或 ./.agents/skills/ 等目录，可随项目提交，团队共用）
npx skills add LeifWebber/git-intent-skills --skill intent-commit

# 或全局安装，对本机所有项目生效
npx skills add LeifWebber/git-intent-skills --skill intent-commit -g
```

CLI 会自动检测你本机安装了哪些 Agent；也可以用 `-a` 指定，例如 `-a claude-code -a codex`。

如果不想额外运行命令，也可以直接把下面这段 prompt 发给你的 AI Agent。类似 Codex 的很多 Agent 本身就有 skill-installer 的内置技能，让它自己完成安装：

```markdown
请安装一个 Agent Skill：把 https://github.com/LeifWebber/git-intent-skills 仓库中 skills/intent-commit 目录（含 SKILL.md 及其同级文件）完整复制到你读取 skills 的目录下（例如 Claude Code 是 ~/.claude/skills/intent-commit/，Codex 是 ~/.agents/skills/intent-commit/）。安装完成后告诉我目录位置，并确认你已能识别名为 intent-commit 的 skill。
```

之后在每个 AI 会话任务完成，你准备提交并且开始一个新的 AI 会话的时候，直接向你的 AI Agent 引用 intent-commit skill 然后发送即可，例如对于 Codex：  

```markdown
$intent-commit
```

或者加上限定范围：  

```markdown
$intent-commit 仅提交与样式优化有关的更改
```

> [!TIP]
> 对于 Parallel Agent：
> 你可能同时启动了多个 Agent 线程进行不同主题的任务，不必担心，此技能只会提交它所属会话相关的内容。甚至对于同一个文件，都只会提交它更改的部分。  

### 安装和使用 intent-history skill

安装方式与上面相同，只是把 skill 名换成 `intent-history`：

```bash
# 安装到当前项目
npx skills add LeifWebber/git-intent-skills --skill intent-history

# 或全局安装
npx skills add LeifWebber/git-intent-skills --skill intent-history -g
```

如果两个都想装，可以一次装完：`npx skills add LeifWebber/git-intent-skills --skill '*'`。

对应的自然语言安装提示词：

```markdown
请安装一个 Agent Skill：把 https://github.com/LeifWebber/git-intent-skills 仓库中 skills/intent-history 目录（含 SKILL.md 及其同级文件）完整复制到你读取 skills 的目录下（例如 Claude Code 是 ~/.claude/skills/intent-history/，Codex 是 ~/.agents/skills/intent-history/）。安装完成后告诉我目录位置，并确认你已能识别名为 intent-history 的 skill。
```

intent-history 的技能是被动触发的，AI 会在执行任务过程中需要的时候主动使用。通常不需要你主动提及使用。不过你依然可以在需要的时候这么做，来代替你手动翻历史和代码注释：  

```markdown
$intent-history 最近一次调整 src/components/ui/popover.tsx 是出于什么目的？对调用处的影响有验证过吗？
```
