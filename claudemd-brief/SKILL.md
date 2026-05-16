---
name: claudemd-brief
description: Make brief CLAUDE.md with official guidance
---

# ClaudeMd brief
## Goal
Make brief CLAUDE.md with claude's official guideline.

## Why
Because claude's official document recommends "brief CLAUDE.md".

> There’s no required format for CLAUDE.md files, but keep it short and human-readable. 
> ...
> CLAUDE.md is loaded every session, so only include things that apply broadly.
> For domain knowledge or workflows that are only relevant sometimes, use skills instead.
> Claude loads them on demand without bloating every conversation.

## Prohibition
* Editing file or directory that not related about CLAUDE.md and skill

## Effective guidance from official
### 1. Include/Exclude to CLAUDE.md

| OK (Include) | NG (Exclude) |
| --- | --- |
| Bash commands Claude can't guess | Anything Claude can figure out by reading code |
| Code style rules that differ from defaults | Standard language conventions Claude already knows |
| Testing instructions and preferred test runners | Detailed API documentation (link to docs instead) |
| Repository etiquette (branch naming, PR conventions) | Information that changes frequently |
| Architectural decisions specific to your project | Long explanations or tutorials |
| Developer environment quirks (required env vars) | File-by-file descriptions of the codebase |
| Common gotchas or non-obvious behaviors | Self-evident practices like "write clean code" |

This can be said in another way that CLAUDE.md should have only 3 points.

* **Specificity**: Is the information claude can't get from the code or general knowledge?
* **Stability**: Will it stay current? If not, link to it instead.
* **Brevity**: Is it a short, actionable instruction? No long explanations or self-evident content

### 2. Adherence
Use "IMPORTANT" or "YOU MUST" to tune instructions by adding emphasis.

### 3. Path syntax
Use `@path/to/file` syntax

```
See @./docs/arch/README.md for architect guidance.
```

### 4. Make as skill
If it's domain knowledge or workflow that are only relevant sometimes, make it as skill. And refer from CLAUDE.md

## Workflow
### 1st step: setup
* [ ] Receive target CLAUDE.md from user
  * **If skill is not passed, out error "Skill is not specified" and stop workflow**
* [ ] Watch around target CLAUDE.md

### 2nd step: pick "exclude"
* [ ] Pick up exclusions referring section "1. Include/Exclude to CLAUDE.md"
* [ ] Group them to "Non-Specificity" / "Non-Stability" / "Non-Brevity"
* [ ] Memory the group for after workflow

### 3rd step: do "exclude"
* [ ] From "Non-Specificity", remove information what claude already know.
  * **Before remove, AskUserQuestion how to remove**
* [ ] From "Non-Stability", put "mutable" info as link.
  * **AskUserQuestion where is intended link**
* [ ] From "Non-Brevity", rewrite it brief without loosing original context.

### 4th step: make as skill
* [ ] If specific section has domain knowledge or workflow, create it as skill
* [ ] After creation, in CLAUDE.md, write reference to skill
  * ex) `use /review-skill for code review`

### 5th step: adhere/path
* [ ] Set adhere "IMPORTANT" and "MUST"
  * AskUserQuestion to user that is it really important or must?
* [ ] Set path with path syntax

## References
* "Write an effective CLAUDE.md"
  * https://code.claude.com/docs/en/best-practices#write-an-effective-claude-md