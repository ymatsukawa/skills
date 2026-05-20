---
name: refactor-standard
description: Refactor code with specific standard.
---

# Refactor standard
## GOAL
Refactoring in clear standard.

## Why
Many developers input prompt just "refactor it".
But this will not satisfy them, because there are not standards.

In order to clear "what to do in minimum", this skill clears standards.

## Philosophy of refactoring
> "Any fool can write code that a computer can understand. Good programmers write code that humans can understand." - Martin Fowler

## Standards
**tidy first**:
Elemental cleanup methods.

Read: `references/tidy_first.md`

**basic standards**:
Standards of refactoring.

Read: `references/basic_standard.md`

## Prohibition
* Self-Judge to operate `git add` and `git commit`

## Workflow

### step-1: Setup
* [ ] Receive where to review. single file? files from diff? around repository?
  * **If not specified any, out error "Target is not specified" and stop workflow.**
* [ ] Scan what to refactor of the target(s)

### step-2: Follow "tidy first"
* [ ] Apply "tidy first" methods to target(s)
  * Run methods one by one because some method partially duplicate content.

### step-3: Follow "basic standards"
* [ ] Apply "basic standards" methods to target(s)
  * Run methods one by one because some method partially duplicate content.
