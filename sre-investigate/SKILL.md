---
name: sre-investigate
description: Investigate SLI/SLO from current system
---

# SRE investigate
## GOAL
From current system components, investigate SLI/SLO standards.

## Why
Many developers want "reliability" for production, but it's vague because of no objective and indicator.

In order to get first step, this skill investigates why/what/how build the reliability.

## Prohibition
* **DO NOT aim 100% reliability** because "perfect" requires insane cost, it's unreal.
* In Editing
  * Editing source code
  * `git add` and `git commit`

## Workflow
### step1: Scan and hunt the target
* [ ] Scan target, default is project.
  * **If no target is specified, out error like "Cannot process" and stop workflow**
* [ ] In scanning, get the unreliability
  * [ ] The standard is at `references/basic-slo-sli-web.md`
  * [ ] If you thought another recommendations, make proposal after step

### step2: Report what should project do
* [ ] After investigation, report format by "SLI/SLO report"
* [ ] Output report as markdown
  * [ ] If user does not specify where to out, AskUserQuestion the path

**SLI/SLO report**:

```markdown
# Current state of SLI/SLO : {System name}

## Critical
{Check List: what is OK/NG}

## High
{Check List: what is OK/NG}

## Medium
{Check List: what is OK/NG}
```

## Meta Info
* This skill is recommended to apply IaaS code; Terraform, CloudFormation, Ansible or etc.