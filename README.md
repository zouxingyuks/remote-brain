# Remote Brain

A shared knowledge repository for [opencode-remote-brain](https://www.npmjs.com/package/opencode-remote-brain).

## Setup

1. Install the plugin:
   ```bash
   npm install opencode-remote-brain
   ```

2. Configure `~/.config/opencode/remote-brain.jsonc`:
   ```jsonc
   {
     "repo": "zouxingyuks/remote-brain",
     "ref": "master"
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

All rules and skills are declared in `index.yaml`. The plugin reads this file as the single source of truth.

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
