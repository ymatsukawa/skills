---
name: golang-avoidance
description: Avoidance guide of golang in developing, refactoring and testing
---

# Golang avoidance
## GOAL
Modify design and code by avoidance guide.

## Why
In many golang project, there are many bias that "It's usual golang way" but the way is danger.

In order to remove the risk, follow guidance and modify design and code.

## Prohibition
* `git add` and `git commit`

## Workflow
### step1: Setup
* [ ] Scan target(s) specified by user
  * **If user does not point any target, out error like "Cannot progress" and stop workflow**

### step2: Scan and understand
* [ ] Scan and understand target file and related modules

### step3: Plan applying guidance
* [ ] With `references/*.md` , plan how to apply design and coding
* [ ] AskUserQuestion that permission to modify

### step4: Modify
* [ ] Modify design and coding with plan
