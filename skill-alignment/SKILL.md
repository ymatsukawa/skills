---
name: skill-alignment
description: Align understanding of skill between AI and user.
---

# Skill alignment
## GOAL
Align understanding of skill between you and user.
If there is mismatch, adjust the skill with user.

## Why
Users have bias that "skill" is "silver bullet".
So, there are many gaps between expectation and result of coding agent.
In order to reduce the gaps, alignment is required.

## Definition
**user**:
Who uses skill.
In this context, who calls you.

**skill**:
Coding agent skill. It's written in SKILL.md

## Prohibition
* Editing file except "skill rule" file or directory
  * skill rule only covers `SKILL.md`, `references/`, `scripts/`, `assets/`

## Setup subagent "newcomer"
Set subagent of **newcomer**.

The subagent has evaluation standard paper that uses after workflow.

```
# Evaluation
I'm agent who execute {skill}

## Target
**Path**:
* (1) {Write skill/references/assets path where I "Read"}
* (2) ...
* (3) ...

## Requirements
* (1) {Write requirement what I understood}
* (2) ...
* (3) ...

## Plan
{Write how I planned}

## Unclear
* (1) {Write what was unclear}
* (2) ...
* (3) ...

## Self-judged
* (1) {Write what "is"/"will be" self-judged}
* (2) ...
* (3) ...
```


## Workflow
**iteration-1**:
* [ ] Receive skill from user
  * **If skill is not specified, out error "skill is not specified", and stop workflow.**
  * **If input is required the skill but not passed, out error "The skill required path/file", and stop workflow.**
* [ ] Launch subagent "newcomer"
* [ ] Confirm agent is new process

**iteration-2**:
* [ ] Execute skill and evaluate by "newcomer"
* [ ] "newcomer" should report as markdown
  * AskUserQuestion where to put file
* [ ] AskUserQuestion where to fix
  * Continue or break is first.

If continue iteration:
* [ ] AskUserQuestion how to modify "Unclear" and "Self-judged"
* [ ] After answer, launch new "newcomer" and pass answer info
* [ ] Close old "newcomer" and iteration-2 is finished.

If break to iteration:
* [ ] Close old "newcomer" and finish iteration-2
* [ ] Notice user that "finished alignment"

## Remarks
* Report output should be UTF-8
  * Default language is "en"
