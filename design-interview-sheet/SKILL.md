---
name: design-interview-sheet
description: Write a design interview sheet through an interview. Run only when the user explicitly calls this skill.
---

# Design interview sheet
## GOAL
Turn what the user wants to design into an interview sheet, through an interview.

## Why
Most instructions given to an AI amount to "just do it nicely".

But "nicely" does not work. It is either too abstract, or the user does not yet know what should be designed.

To avoid this, write an interview sheet and ask the user questions until "what should be done" is clear.

## Your role
Present the designs that follow from the user's request, split into a recommended one and the rest.

The workflow ends when the sheet is written. Filling in `### Decision` and acting on it are out of scope: do not wait for the user's checkmarks, do not read them back, do not implement or fix.

## Prohibition
* `git add`, `git commit`
* Editing files other than the specified one

## Workflow
### step1: Setup
- [ ] Receive from the user what they want to design
  - **If the user gives no concrete instruction, raise an error such as "cannot proceed" and stop the workflow**
- [ ] Understand what is being designed and its target

### step2: Question about obscureness
- [ ] Ask the user with AskUserQuestion about what is unclear to you: the background knowledge and the material you need to judge the "design questions"
  - ATTENTION: these are not step3's "questions for the design" but "questions to remove what you do not know"
- [ ] Cover everything the design needs

### step3: Write interview sheet
- [ ] Write the sheet in the "interview sheet frame" format
  - [ ] If no output path is given, ask the user with AskUserQuestion where to write it
- [ ] Report the output path, then stop

"interview sheet frame":

```markdown
# Design about {title}
## Design-1: {title}
### Target
- {target of design}
- ...

### Why
- {reason of design in brief}
- ...

### Risk of not doing
- {risk of not doing in brief}
- ...

### Plans
**Plan-A(Recommended)**:
{Detail of plan-A}

**Plan-B**:
{Detail of plan-B}

**Why Recommended Plan-A**:
{reason to take plan-A}

### Decision
* [ ] Plan-A
* [ ] Plan-B
* [ ] Another way
  * __WRITE_HERE__

## Design-2: {title}
...
```
