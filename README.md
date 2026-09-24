# Intent Skills

[English](README.en.md)

让 Git 提交留下可核实的设计意图，让后续维护能够找到并检验这些意图。

这两个 Agent 技能覆盖提交的编写与历史的使用：

| 技能 | 何时使用 | 产出 |
| --- | --- | --- |
| [intent-commit](skills/intent-commit/SKILL.md) | 拟写或审查提交信息、规划拆分、按授权创建提交 | 按独立意图组织的提交，记录关键决策、真实验证和维护边界 |
| [intent-history](skills/intent-history/SKILL.md) | 排查回归、解释特殊逻辑、评估兼容约束或修改既有行为 | 有提交与代码依据的解释，区分仍有效的约束、过时实现和证据缺口 |

两个技能可以单独安装。`intent-history` 可以读取普通提交，不要求历史由 `intent-commit` 生成；提交记录越充分，能够恢复的上下文通常越多。

## 设计原则

- **以独立意图组织提交。** 同一行为涉及的代码、测试与配置通常一起提交，不按目录机械拆分。
- **保留 diff 无法解释的信息。** 写明已确认的动机和关键取舍，不事后编造决策过程。
- **验证记录有明确范围。** 实际运行了什么就记录什么，不将工作区验证说成每个中间提交都已验证。
- **历史需要核对。** 完整提交信息与 diff、当前代码和后续变更相互印证；过去的验证不等于现在已通过。
- **保护用户已有工作。** 拟写信息不操作 Git；提交需明确授权，不能混入无关改动或自动推送。

## 语言与项目约定

技能正文以中文维护，生成的提交信息可以使用其他语言。

语言和格式依次参考用户明确要求、目标项目规则、近期稳定惯例；没有约定时，语言沿用用户当前交流语言，格式默认采用 [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)。目标项目已有规范可以覆盖默认格式，项目强制检查发生冲突时需要先解决冲突。

“实现与决策”“验证”“边界与后续”是可选的信息组织方式，不是必须填满的表单。简单改动可以只用标题。

## 安装

技能采用 [Agent Skills 格式](https://agentskills.io/specification)，不包含运行时脚本，也不要求特定技术栈、MCP 服务或 Entire。执行提交和检索历史需要 Agent 能使用 Git 并访问目标仓库；只根据提供的材料拟写信息时不要求仓库访问。

获取本仓库文件后，将 `skills/intent-commit/` 和／或 `skills/intent-history/` **整个目录**复制到所用工具的技能目录。保留技能目录中的 `LICENSE`。每个目录都是独立的安装单元，不需要把整个仓库或外层 `skills/` 再嵌套一层。

### Codex

依据 [OpenAI 官方技能文档](https://learn.chatgpt.com/docs/build-skills)，用户级目录为 `~/.agents/skills/`，项目级目录为目标项目的 `.agents/skills/`。

在本仓库根目录执行以下命令，可为当前用户首次安装两个技能（macOS、Linux 或兼容 shell）。遇到任一同名目录或符号链接时停止，避免覆盖已有版本：

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

项目级安装时，将 `intent_skills_dir` 改为目标项目的 `.agents/skills` 绝对路径。也可以通过文件管理器复制；只安装一个技能时只复制对应目录。更新前先比较并备份已有版本，避免覆盖本地定制。用户级与项目级的同名技能不会自动合并，建议为同一使用场景保留一个版本。

Codex 通常会自动发现变更，未显示时重启。`agents/openai.yaml` 仅提供 Codex 展示名称与示例提示，不关闭自动发现，也不授予写入权限。

### 其他 Agent

将完整技能目录放入该工具官方文档指定的位置。`SKILL.md` 使用通用格式，`agents/openai.yaml` 是可选的产品元数据。技能发现、调用语法和执行权限由具体宿主决定；本项目不声称已在所有宿主上完成运行验证。

## 使用

在支持 `$技能名` 的宿主中可以使用下面的提示；其他宿主通过其技能选择界面或对应调用语法选择技能。

只拟写与规划：

```text
$intent-commit 为这次缓存修复拟写提交信息；如需拆分，请解释理由。先不要暂存或提交。
```

创建提交：

```text
$intent-commit 把本次任务确认的缓存修复及对应测试整理并提交，保留其他改动，不推送。
```

读取历史：

```text
$intent-history 查明 src/cache.ts 为什么保留这个 fallback，结合后续提交和当前测试判断它是否仍有必要。
```

历史检索本身不修改文件。没有足够证据时，Agent 应说明无法确认的部分，而不是把猜测写成历史事实。更多边界示例见 [使用示例](docs/examples.md)。

## 维护与验证

本仓库以技能指令为交付物，没有需要安装的项目运行依赖。修改技能时，核对触发条件、授权范围、语言与格式覆盖、链接及许可证，并在隔离仓库中验证受影响的行为。参见 [维护与验证指南](CONTRIBUTING.md)。

本文档中的使用场景是行为说明，不是跨模型、跨宿主的通过率报告。

## 许可证

[MIT](LICENSE) © 2026 LeifWebber。每个技能目录附带同一许可证，方便独立分发。
