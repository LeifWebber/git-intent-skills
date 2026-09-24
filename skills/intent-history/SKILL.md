---
name: intent-history
description: 从 Git 提交信息及相关 diff 中恢复历史动机、设计决策、验证记录和边界约束。当排查回归、解释特殊逻辑、修改既有行为或判断兼容约束需要历史依据时使用；兼容普通提交，不要求特定提交格式。
license: MIT
---

# 从意图式提交中恢复设计上下文

本仓库的部分 Git commit message 经过刻意维护，其信息密度和历史价值高于普通仓库。因此，在理解历史行为和设计原因时，Git history 中的完整提交信息应被视为与代码、测试和文档并列的重要上下文来源。

## 何时追溯

不要为了例行流程而无差别读取 Git history。

当历史上下文可能影响当前判断时，应主动查询。特别是：

- 排查 bug、regression 或“以前正常、现在异常”的行为；
- 修改已有行为、数据模型、生命周期、权限、公共 API 或兼容逻辑；
- 遇到看似多余、奇怪、复杂或可以简化的代码；
- 准备删除 workaround、fallback、兼容代码或特殊分支；
- 测试要求某个行为成立，但当前代码无法解释原因；
- 需要判断某段代码是偶然实现细节还是刻意设计；
- 修改可能影响既有数据身份、历史数据、序列化格式或数据库记录的逻辑；
- 当前需求与已有实现明显冲突，需要了解原设计目的。

在这些场景中，不要仅根据当前代码猜测历史原因。优先通过 Git history 验证。

## 定位相关提交并读取完整上下文

优先进行局部、目标明确的历史检索，而不是阅读整个 git log。

一些可能的检索方式如下，根据问题选择合适方式，不必不机械遍历所有示例：

```bash
git --no-pager log -n 20 --oneline -- <path>
git --no-pager log -n 20 --format=fuller --grep='<concept>'
git --no-pager log -n 20 -S'<symbol-or-string>' -- <path>
git --no-pager log -n 20 -G'<regex>' -- <path>
git --no-pager blame -L <start>,<end> -- <path>
git --no-pager show --format=fuller --stat <commit>
git --no-pager show --format=fuller <commit> -- <path>
```

通过项目历史的 git commit msg 确认项目提交使用的语言，来确认搜索时用到的关键词的语言。

`--oneline` 仅用于定位。找到候选提交后，读取完整 message 和相关 diff；需要理解跨文件机制时再扩大 diff 范围。检查后续修复、迁移和 revert，避免将已被替代的设计当作当前约束。文件改名时可用 `git log --follow -- <path>` 继续追溯。

若 Git 记录仍不足，可读取已经可访问的相关 PR、issue、设计文档或会话记录。
若本机已安装 Entire CLI 并且你确实需要用到 Entire 工具的时候，**仅在找到相关 commit 后**，再通过 commit msg footer 的 Entire-Checkpoint 来结合 entire checkpoint explain 补充原始讨论。

没有相关记录或工具不可用时，明确证据缺口，不编造背景。

## 按语义解读提交信息

项目里 git commit msg 很可能采用了这种结构：

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

## 如何使用历史信息

历史 commit 是重要证据，但不是不可推翻的规范。

明确区分三类内容：

- 仍然成立的设计不变量和数据约束；
- 当时环境下的实现选择；
- 已经过时的限制或 workaround。

如果当前修改会违反历史 commit 中明确记录的不变量、兼容性要求或数据约束，应在修改前确认这种变化是当前任务有意要求的，而不是无意回归。

尤其不要仅仅因为某段逻辑“现在看起来可以简化”就删除它。如果历史记录表明它用于避免某个具体 regression、数据损坏、兼容性问题或边界条件，应保留这一约束，或者以新的方式明确解决原问题。
