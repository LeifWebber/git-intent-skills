<h3 align="center">Intent-based git commits, and how to consume them</h3>

<p align="center">
Turn your git history into an asset that makes a decisive difference for AI agents
</p>

<p align="center">
  <a href="README.md"><strong>简体中文</strong></a> ·
  <span><strong>English</strong></span>
</p>

<br/>

## Why this project exists

Efficient, precise context is critical for an AI agent to finish a task well.

While working on a task, an agent may run several rounds of agentic search to gather what it needs. Reaching the one paragraph in the one source file that actually solves the problem usually takes multiple rounds, and every round fills precious context with irrelevant material. Even then, the AI only sees the surface of the code. Unlike a human engineer, it can rarely see the design intent behind it.

That is why people maintain "project docs / knowledge bases" for AI, or do things like "capturing lessons learned", so the AI can reuse existing experience and background when it hits a problem instead of exploring an ocean of source code.

Codex and Claude both shipped memory-related features this February.
Codex, for example, stores summaries, persistent entries, recent inputs and supporting evidence from earlier chats under `~/.codex/memories/`. See the [official Codex Memories docs](https://learn.chatgpt.com/docs/customization/memories?surface=cli#cli-local-memory-storage).

Another similar tool is [Entire](https://entire.io/), which keeps the full AI session record on every commit and lets you search it semantically when you need it.

My approach is simpler: use the git commit message itself.
I have practiced this for a long time in the enterprise projects I maintain, and I regularly see it hand the AI a crucial piece of information at exactly the right moment. It also has a few advantages over the other approaches that I did not expect:

1. No extra CLI / MCP / hook to install, so it works with any AI harness.
2. Zero configuration, easy to use and easy to understand. You get a high-quality project knowledge base without building a complex knowledge-capture workflow.
3. Almost no intrusion into your team's daily workflow. Teammates who have not installed the skill benefit too, because the commit messages live in the repository and anyone, human or agent, can read them.

## What it is and what it does

This project contains just two skills that follow the [Agent Skills specification](https://agentskills.io/specification). There are no runtime scripts, and no particular tech stack or MCP server is required.

### 1. [intent-commit skill](skills/intent-commit/SKILL.md), the producer

A session between you and an AI usually revolves around one topic (stuffing unrelated tasks into a single AI session is not recommended anyway). When the task in that session is done, you invoke the intent-commit skill once, by hand.

The AI takes **only the changes related to this session** and splits them into one or more git commits that follow the [Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/). Every commit message is carefully designed and usually takes this shape:
```text
<type>(<scope>): <purpose, fix, or behavior change>

<necessary background and motivation; omitted when there is nothing to add>

Implementation and decisions:
- <key approach, mechanism, or evidence-based trade-off>

Verification:
- <checks actually performed, their results, and what they cover>

Boundaries and follow-ups:
- <important constraints, uncovered scope, risks, or required follow-up work>

<optional Conventional Commits footer>
```

Here is the thinking behind each part:

#### Recording intent

Before the vibe-coding era, commit messages tended to say "what was done this time". intent-commit messages focus on "**what this change was trying to do**", that is, the intent behind the change.

I designed it this way because in the vibe-coding era, the real "source" of a program is no longer the programming language. It is the prompts that guided the AI through building the project step by step, together with the architectural decisions, technology choices and trade-offs contained in those prompts.

That information is useful not only to the AI that will work on the project later, but also to humans reading the commit history. I find myself reading source code less and less these days. What I want to know instead is "why was it done this way back then", "what line of thinking guided this change, and what key decisions were made".

For an AI, seeing the intent and background of a file's past changes while viewing its history lets it maintain your project and write code more like an architect, instead of being stuck in local optima and producing endless patches.

#### Recording verification

In projects built and maintained with AI, explicit verification and testing are becoming more important and more common.

Your agent has very likely already run various tests during the session. Having it record them on the way out is highly valuable for future AI troubleshooting, because the AI can then ask:

> What behavior did this commit **claim to guarantee** back then?

And when a test breaks later, it can quickly notice:

> "This exact path was specifically verified back then, and now it is broken again."

#### Recording impact and follow-ups

In this part, the AI records what impact this change may have on the system, the key trade-offs made during implementation, and anything else that future maintainers or AI agents should be aware of.

This plays a key role later, when an AI agent investigates a regression, explains an odd piece of logic, modifies existing behavior, or judges a compatibility constraint.

### 2. [intent-history](skills/intent-history/SKILL.md) skill, the consumer

This skill guides the AI in consuming the intent-style commit messages produced by intent-commit.

It makes the AI look up git history at the right moment and in the right way, aiming to maximize the value intent-commit brings to the task at hand.

Consumption mainly takes two forms:

1. Semantic keyword search

The AI searches past commit messages by keyword to locate the key source files while picking up the intent-style commit messages at the same time. The effect is somewhat similar to the [entire search command](https://docs.entire.io/cli-reference/search#search).

2. Tracing the history of a file or folder

This usually happens once the AI has already located the key source file. It then pulls the intent-commit messages related to that file to understand the background.

## Installation and usage

> [!note]
> Installing both skills right away is not recommended, because your project's existing commit history probably does not yet meet what the intent-history skill needs.
> 
> Install intent-commit first, use it to accumulate enough intent commits, and then add intent-history.

### Install and use the intent-commit skill

You can install it into almost every mainstream AI agent with a single [skills CLI](https://github.com/vercel-labs/skills) command:

```bash
# Install into the current project (written to ./.claude/skills/, ./.agents/skills/ etc.; commit it so the whole team shares it)
npx skills add LeifWebber/git-intent-skills --skill intent-commit

# Or install globally, for every project on this machine
npx skills add LeifWebber/git-intent-skills --skill intent-commit -g
```

The CLI auto-detects which agents are installed on your machine. You can also pick them explicitly with `-a`, for example `-a claude-code -a codex`.

If you would rather not run a command, paste the following prompt to your AI agent. Many agents, Codex among them, ship with a built-in skill-installer skill, so just let it do the installation itself:

```markdown
Please install an Agent Skill: copy the skills/intent-commit directory (SKILL.md and its sibling files) from the repository https://github.com/LeifWebber/git-intent-skills into the directory you load skills from (for example ~/.claude/skills/intent-commit/ for Claude Code, or ~/.agents/skills/intent-commit/ for Codex). When done, tell me the directory you used and confirm that you can now recognize a skill named intent-commit.
```

Then, whenever a session's task is finished and you are about to commit and start a new AI session, simply reference the intent-commit skill in your message. For Codex, for example:

```markdown
$intent-commit
```

Or narrow the scope:

```markdown
$intent-commit only commit the changes related to the style tweaks
```

> [!note] Parallel agents
> You may be running several agent threads on different topics at the same time. Don't worry: this skill only commits the content related to its own session. Even within a single file, it only commits the parts that session changed.

### Install and use the intent-history skill

Installation is the same as above, just with the skill name changed to `intent-history`:

```bash
# Install into the current project
npx skills add LeifWebber/git-intent-skills --skill intent-history

# Or install globally
npx skills add LeifWebber/git-intent-skills --skill intent-history -g
```

To install both at once: `npx skills add LeifWebber/git-intent-skills --skill '*'`.

The matching natural-language install prompt:

```markdown
Please install an Agent Skill: copy the skills/intent-history directory (SKILL.md and its sibling files) from the repository https://github.com/LeifWebber/git-intent-skills into the directory you load skills from (for example ~/.claude/skills/intent-history/ for Claude Code, or ~/.agents/skills/intent-history/ for Codex). When done, tell me the directory you used and confirm that you can now recognize a skill named intent-history.
```

The intent-history skill is triggered passively: the AI uses it on its own when a task calls for it, so you normally don't need to mention it. You can still invoke it explicitly whenever you like, as a replacement for digging through history and code comments yourself:

```markdown
$intent-history What was the purpose of the most recent change to src/components/ui/popover.tsx? Was its impact on the call sites verified?
```
