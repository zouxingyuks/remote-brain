# Remote Brain

A shared knowledge repository for [opencode-remote-brain](https://www.npmjs.com/package/opencode-remote-brain).

## Structure

```
index.yaml                          # Entry point — declares rules and skills
rules/                              # Markdown files injected as OpenCode instructions
  agent-rules.md                    # Tool usage & language conventions
skills/                             # Skill directories (each must contain SKILL.md)
  superspec-init/SKILL.md           # Project initialization
  superspec-research/SKILL.md       # Research & analysis
  superspec-plan/SKILL.md           # Planning & design
  superspec-implement/SKILL.md      # Implementation
```

## Usage

1. Install the plugin:
   ```bash
   npm install opencode-remote-brain
   ```

2. Configure `~/.config/opencode/remote-brain.jsonc`:
   ```jsonc
   {
     "repo": "zouxingyuks/remote-brain",
     "ref": "main"
   }
   ```

3. Register in your OpenCode `config.json`:
   ```json
   {
     "plugins": {
       "remote-brain": "opencode-remote-brain"
     }
   }
   ```

## Content

### Rules

| File | Description |
|------|-------------|
| `rules/agent-rules.md` | Serena, Context7, sequential thinking 工具使用规范；中英文语言约定 |

### Skills

| Skill | Description |
|-------|-------------|
| `superspec-init` | 项目初始化 |
| `superspec-research` | 调研与分析 |
| `superspec-plan` | 规划与设计 |
| `superspec-implement` | 实现与编码 |

## Adding Content

### Rules

Create a `.md` file in `rules/` and add its path to `index.yaml`:

```yaml
rules:
  - rules/my-new-rule.md
```

### Skills

Create a directory in `skills/` with a `SKILL.md` inside, then add it to `index.yaml`:

```yaml
skills:
  - skills/my-new-skill
```
