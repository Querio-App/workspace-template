---
name: create-skill
description: Create a new skill for the workspace. Skills are markdown instruction files that the agent loads on-demand for specialised tasks.
---

# Creating Skills

Skills are markdown files at `skills/<name>/SKILL.md` with YAML
frontmatter. They provide specialised instructions the agent loads when
a task matches the skill description.

The `skills/` paths below are relative to `[context].root` in the workspace-root `querio.toml` (default `.`). Use the resolved skills directory from the system prompt; do not assume it is at the workspace root.

## How Skills Work

1. At startup, skill names and descriptions are listed in the system prompt
2. When a task matches, read the full SKILL.md to get detailed instructions
3. Skills can also be invoked explicitly with `/<name> <args>`

This is progressive disclosure — only descriptions are always in context,
full instructions load on-demand.

## Skill Structure

```
skills/
└── my-skill/
    ├── SKILL.md          # Required: frontmatter + instructions
    ├── scripts/          # Optional helper scripts
    │   └── setup.sh
    └── references/       # Optional detailed docs
        └── api.md
```

## SKILL.md Format

```markdown
---
name: my-skill
description: What this skill does and when to use it. Be specific.
---

# My Skill

Instructions for the agent go here. Use markdown.

## Setup

Run once before first use:
\`\`\`bash
pip install some-package
\`\`\`

## Usage

Describe the workflow step by step.
```

## Frontmatter

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Lowercase a-z, 0-9, hyphens. Should match directory name. |
| `description` | Yes | What the skill does and when to use it. |

### Description Best Practices

The description determines when the agent loads the skill. Be specific.

Good:
```yaml
description: Extracts and analyses data from PDF files using tabula-py. Use when working with PDF documents or scanned tables.
```

Poor:
```yaml
description: Helps with PDFs.
```

## Rules

- Skill directory name must match the `name` in frontmatter
- Skills without a description are not loaded
- The SKILL.md file is the only required file
- Use relative paths to reference scripts and assets within the skill directory
- Heavy setup (pip installs, API keys) should go in a Setup section
- Keep instructions actionable — tell the agent what to do, not what the skill is
- After creating a skill, run `/reload` to make it available

## Example: Data Quality Skill

```
skills/data-quality/
├── SKILL.md
└── scripts/
    └── check.py
```

**SKILL.md:**
```markdown
---
name: data-quality
description: Run data quality checks on tables. Use when asked to validate, audit, or check data quality.
---

# Data Quality Checks

## Workflow

1. Ask which table to check (or use the one mentioned)
2. Run the check script to get a quality report
3. Present findings and suggest fixes

## Running Checks

Set `skill_dir` to the directory containing this SKILL.md, including the configured context-root prefix.

\`\`\`python
import subprocess
result = subprocess.run(
    ["python", "scripts/check.py", table_name],
    cwd=skill_dir, capture_output=True, text=True
)
print(result.stdout)
\`\`\`

## What to Check

- Null rates per column (flag > 5%)
- Duplicate rows
- Column type mismatches
- Outliers (values > 3 std devs from mean)
- Referential integrity against related tables
```
