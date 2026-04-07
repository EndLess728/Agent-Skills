# Agent Skills Repository

This repository is for creating and maintaining custom agent skills.

## What Is A Skill?

A skill is a focused capability packaged as a folder containing a `SKILL.md` file.
The `SKILL.md` includes:

- Metadata (`name`, `description`)
- Clear instructions for when and how the skill should be used
- Optional examples and migration/checklist steps

## Repository Structure

```text
Agent-Skills/
├── README.md
└── <Skill-Folder-Name>/
    └── SKILL.md
```

Current example:

- `Migrate-Unistyles-From-V2-V3/SKILL.md`

## Recommended Naming

- Folder name: descriptive, title-style with hyphens
  - Example: `Migrate-Unistyles-From-V2-V3`
- Skill `name` (inside frontmatter): short, lowercase, machine-friendly
  - Example: `unistyles-v2-to-v3`

## Skill Template

Create a new folder and add `SKILL.md` using this template:

````md
---
name: your-skill-name
description: One-line summary of what this skill does and when to use it.
---

# Skill Title

Short intro describing the purpose.

## When To Use

- Trigger 1
- Trigger 2
- Trigger 3

## Steps

### Step 1

What to do.

### Step 2

What to do next.

## Example

```ts
// Minimal example (optional)
```

## Notes

- Constraints, caveats, or common pitfalls.
````

## Workflow For Adding A New Skill

1. Create a new folder in the repo root.
2. Add a `SKILL.md` file using the template above.
3. Keep instructions concrete, testable, and task-oriented.
4. Include examples where ambiguity is possible.
5. Commit with a clear message.

## Quality Checklist

- Frontmatter has both `name` and `description`.
- The scope is narrow and specific (not too broad).
- Trigger conditions are explicit.
- Steps are actionable and ordered.
- Examples match the target tech stack.

## Future Enhancements (Optional)

- Add a script to validate frontmatter fields across all skills.
- Add a lint/check workflow for consistent formatting.
- Add a catalog table listing all skills and their purpose.

## Reference

- [Anthropic Skill Creator Example](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)
