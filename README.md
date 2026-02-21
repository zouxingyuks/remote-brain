# Remote Brain

A shared knowledge repository for [opencode-remote-brain](https://www.npmjs.com/package/opencode-remote-brain).

## Structure

```
index.yaml              # Entry point — declares rules and skills
rules/                  # Markdown files injected as OpenCode instructions
  example-rule.md
skills/                 # Skill directories (each must contain SKILL.md)
  example-skill/
    SKILL.md
```

## Usage

1. Install the plugin:
   ```bash
   npm install opencode-remote-brain
   ```

2. Configure `~/.config/opencode/remote-brain.jsonc`:
   ```jsonc
   {
     "repo": "your-org/remote-brain",
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
