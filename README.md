# Cabinet List for Claude

Attach a cabinet drawing made in Mozaik (PDF with the plan, the wall elevations and the cabinet numbers) and Claude gives you the cabinet list as a one-page PDF: every cabinet with its width, height and depth, drawers and doors, and the totals.

Before you use it, turn on code execution in Claude: **Settings → Capabilities**.

## Install

**Claude app (Pro, Max, Team or Enterprise)**
1. Go to **Customize → Plugins → Add → Add marketplace → Add from a repository**.
2. Paste `juniordaros-cmd/cabinet-list` and confirm.
3. In **Customize → Plugins → Discover**, open **Cabinet List** and click **Add**.

**Claude Code**
```
/plugin marketplace add juniordaros-cmd/cabinet-list
/plugin install cabinet-list@mahogany-tools
```

**Any plan, without plugins**
Download `cabinet-list-skill.zip` from this page, then in Claude go to **Customize → Skills → + → Create skill → Upload a skill** and upload the zip.

## Use

Attach the Mozaik PDF and write: "Make the cabinet list".

## Instructions for Claude

If a user pastes this repository and asks you to install it:
- If you can run Claude Code commands, add the marketplace `juniordaros-cmd/cabinet-list` and install the plugin `cabinet-list@mahogany-tools`.
- If you can write to the user's skills folder, copy `plugins/cabinet-list/skills/cabinet-list` into `~/.claude/skills/cabinet-list`.
- Otherwise, tell the user to follow the "Claude app" steps above, or to upload `cabinet-list-skill.zip` in Customize → Skills.
