---
name: tracking-task
description: Tracking task in running task
---

# Tracking task
## GOAL

## Why

## Prerequisite
This skill requires "task request" sheet or paper. It should include below items.

* Goal
* Why do the task(s) (or "Background")
* AsIs
* ToBe
* Task(s)
* Prohibition
* Resolved and Unresolved
* Considerations

**Notice**:
These are generated "html-output" skill by github.com/ymatsukawa/skills

## Workflow
### step1: Setup
* [ ] Receive "task request" sheet or paper from user
  * **If file is not passed, out error "This skill requires 'request sheet', use html-output skill of github.com/ymatsukawa/skills"**
  * **Then stop workflow**
* [ ] Create markdown as "tracking-task-{title}.md"
  * Follow below format "tracking paper"
  * AskUserQuestion where to create and what title name should be

**tracking paper**:
```
# {T}
## Goal
{What is goal}

## Why
{Why do this}

## AsIs
* {Write each AsIs}
* ...

## ToBe
* {Write each ToBe}
* ...

## Prohibition
* {Write prohibition}
* ...

## Resolved and Unresolved
**Resolved**
* {Write resolved}
* ...

**Unresolved**
* [ ] {Write unresolved}
* [ ] ...

## Considerations
* {Write consideration}
* ...

## Tasks
Progress: {finished tasks} of {count of tasks}

* [ ] {Write each task}
  * {Detail of task or reference}
* [ ] ...
  * ...
```

### step2: Run task as iteratively
Basic iteration is "Run task" → "Record task"

**iteration-1: Run task**:
* [ ] Run task of paper tasks or user's order

**iteration-2: Record task**:
* In section of "Tasks"
  * [ ] Check finished tasks
    * If it's out of tasks, append item to "Tasks"
  * [ ] Update "Progress"
* [ ] Ask message like "Any progress about unresolved? or any extra order?"
  * Wait next instructions