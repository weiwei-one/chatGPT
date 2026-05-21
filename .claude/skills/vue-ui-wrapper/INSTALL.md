# Install vue-ui-wrapper Skill

This zip is intended to be extracted as a skill directory named `vue-ui-wrapper`.

## Claude Code: global user skill

Install globally for your own Claude Code usage:

### macOS / Linux

```bash
mkdir -p ~/.claude/skills
unzip vue-ui-wrapper.zip -d ~/.claude/skills
```

After extraction, the file should exist here:

```txt
~/.claude/skills/vue-ui-wrapper/SKILL.md
```

### Windows PowerShell

```powershell
New-Item -ItemType Directory -Force "$HOME\.claude\skills"
Expand-Archive -Path .\vue-ui-wrapper.zip -DestinationPath "$HOME\.claude\skills" -Force
```

After extraction, the file should exist here:

```txt
$HOME\.claude\skills\vue-ui-wrapper\SKILL.md
```

Restart Claude Code after installation.

## Claude Code: project skill

Install into a single project:

```bash
mkdir -p .claude/skills
unzip vue-ui-wrapper.zip -d .claude/skills
```

Expected path:

```txt
.claude/skills/vue-ui-wrapper/SKILL.md
```

## Trigger examples

```txt
Use the vue-ui-wrapper skill to create a BaseInput component.
```

```txt
使用 vue-ui-wrapper skill 检查这个 BaseDialog 的 props、emits、attrs、slots、v-model、expose 设计。
```
