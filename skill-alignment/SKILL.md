---
name: skill-alignment
description: Align understanding of skill between AI and user.
---

# Skill alignment
## GOAL
* Align understanding of skill between you and user.
* If there is mismatch, adjust the skill with user.

## Why
Users have bias that "skill" is "silver bullet".
So, there are many gaps between expectation and result of AI.
In order to reduce the gaps, alignment is required.

## Definition
**user**:
Who uses skill.
In this context, who calls you.

**skill**:
Skill of AI. It's written in SKILL.md

## Prohibition
* Editing file except "skill rule" file or directory
  * skill rule only covers `SKILL.md`, `references/`, `scripts/`, `assets/`

## Setup subagent "newcomer"
Set subagent of **newcomer**.

The subagent has evaluation standard paper that uses after workflow.

```
# Evaluation
I'm agent who run {skill}

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
  * {Write reason why it's unclear}
* (2) ...
  * ...
* (3) ...
  * ...

## Self-judged
* (1) {Write what "is"/"will be" self-judged}
  * {Write reason why "is"/"will be" self-judged}
* (2) ...
  * ...
* (3) ...
  * ...
```


## Workflow
**iteration-1**:
* [ ] Receive skill from user
  * **If skill is not specified, out error "Skill is not specified", and stop workflow.**
  * **In case skill requires "input" but not passed, out error "The skill required path/file", and stop workflow.**
* [ ] Launch subagent "newcomer"
* [ ] Confirm the agent is new process

**iteration-2**:
* [ ] Execute and evaluate the skill by "newcomer"
* [ ] Report by "newcomer" with standard paper
  * AskUserQuestion where to put file
* [ ] AskUserQuestion whether fix/modify is required after watched report
  * Question: "continue or break"

If "continue" iteration:
* [ ] AskUserQuestion how to fix/modify "Unclear" and "Self-judged"
* After fix/modify:
  * [ ] Do "Close current subagent process"
  * [ ] Step back to iteration-1

If "break" iteration:
* [ ] Do "Close current subagent process"
* [ ] Notice user that "Finished alignment"

**Close current subagent process**:
* [ ] Close current "newcomer"
* [ ] Confirm the agent process is closed

## Remarks
* Report output should be UTF-8
  * Default language is "en"
