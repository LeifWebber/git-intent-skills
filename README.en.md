# Intent Skills

[中文](README.md)

Preserve verifiable design intent in Git commits, then recover and check that intent during maintenance.

| Skill | Use it for | Result |
| --- | --- | --- |
| [intent-commit](skills/intent-commit/SKILL.md) | Drafting or reviewing messages, planning atomic commits, and creating authorized commits | Changes grouped by intent, with real decisions, verification results, and maintenance constraints |
| [intent-history](skills/intent-history/SKILL.md) | Investigating regressions, explaining unusual code, and evaluating existing behavior or compatibility constraints | Evidence from commits and code, distinguishing current constraints from obsolete choices and missing context |

Install either skill independently or use both. `intent-history` works with ordinary commits; it does not require history produced by `intent-commit`.

## Principles

- Group changes by independent intent, rather than by file, directory, or technical layer.
- Preserve context that the diff cannot explain. Do not invent motivations or rejected alternatives after the fact.
- Record only completed verification and its actual scope. Testing the full working tree does not prove every intermediate commit was tested.
- Check historical claims against diffs, current code, and later changes. Historical test results do not establish current correctness.
- Preserve existing work. Drafting does not authorize Git writes; committing requires an explicit request and does not authorize pushing.

## Language and conventions

The skill instructions are maintained in Chinese. Generated messages can use other languages.

Language and format follow the user's explicit request, the target project's rules, then stable recent conventions. When none apply, use the user's conversation language and default to [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/). Existing project conventions can override that default. Resolve conflicts with mandatory project checks before proceeding.

Sections for decisions, verification, and boundaries are optional. Do not fill empty sections or pad a simple change with a long body.

## Installation

The skills use the [Agent Skills format](https://agentskills.io/specification). They contain no runtime scripts and require no particular application stack, MCP server, or Entire installation. Committing and history queries require Git access to the target repository. Drafting from supplied evidence does not.

After obtaining this repository, copy the **complete** `skills/intent-commit/` and/or `skills/intent-history/` directory into your host's skill location. Include each skill's `LICENSE`. Each directory is an independent installation unit; do not nest the repository or the outer `skills/` directory inside it.

### Codex

The [official OpenAI documentation](https://learn.chatgpt.com/docs/build-skills) lists `~/.agents/skills/` for user skills and `.agents/skills/` in the target repository for project skills.

Run this from the repository root for a first-time user installation in macOS, Linux, or a compatible shell. It stops if either destination already exists, including a symbolic link:

```sh
(
  set -eu
  intent_skills_dir="$HOME/.agents/skills"
  for intent_skill in intent-commit intent-history; do
    if [ -e "$intent_skills_dir/$intent_skill" ] || [ -L "$intent_skills_dir/$intent_skill" ]; then
      printf 'Already exists: %s\n' "$intent_skills_dir/$intent_skill" >&2
      exit 1
    fi
  done
  mkdir -p "$intent_skills_dir"
  cp -R skills/intent-commit skills/intent-history "$intent_skills_dir/"
)
```

For project installation, set `intent_skills_dir` to the absolute path of the target project's `.agents/skills` directory. File-manager copying also works; copy only one directory if you want one skill. Before updating, compare and back up existing versions to preserve local customizations. Skills with identical names at user and project scopes are not merged; prefer one version for a given use case.

Codex normally detects changes automatically; restart if a skill does not appear. The optional `agents/openai.yaml` files provide display metadata and example prompts. They do not disable implicit discovery or authorize writes.

### Other agents

Use the skill location documented by your host. `SKILL.md` is portable; `agents/openai.yaml` is optional product metadata. Discovery, invocation syntax, and execution permissions depend on the host. This project does not claim runtime testing in every host.

## Usage

These examples use `$skill-name` syntax. In other hosts, use the skill picker or the equivalent invocation syntax.

Draft only:

```text
$intent-commit Draft an English commit message for this cache fix and explain any proposed split. Do not stage or commit.
```

Create commits:

```text
$intent-commit Commit the cache fix and tests confirmed in this task. Preserve unrelated changes and do not push.
```

Investigate history:

```text
$intent-history Find why src/cache.ts retains this fallback, then check later commits and current tests to determine whether it is still needed.
```

History investigation does not modify files. When evidence is missing, report the gap rather than inventing historical facts. See [examples](docs/examples.md) and the [contribution and validation guide](CONTRIBUTING.md), both maintained in Chinese.

## Maintenance and license

This repository ships instructions and has no project runtime dependencies. Check discovery boundaries, authorization, convention overrides, links, and licenses when editing; validate affected behavior in an isolated repository. Example scenarios are guidance, not benchmark results across models or hosts.

[MIT](LICENSE) © 2026 LeifWebber. Each skill includes the same license for independent distribution.
