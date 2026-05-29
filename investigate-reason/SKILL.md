---
name: investigate-reason
description: Investigate root reason of problem.
---

# Investigate reason
## GOAL
Investigate root reason of problem.

## Why
When staying on the same project for a long time, developers start making assumptions.

In order to avoid the assumptions, this skill begins "zero-based" investigation.

## Prohibition
* Editing source files.
* `git add` and `git commit`

## Workflow
### step1: Receive what user want to resolve
* [ ] Receive sheet or paper that user what to resolve
  * **If user does not specify the input, out error like "Cannot start skill because any info about problem", and stop workflow.**

**Strongly recommended**:
Sheet or paper has "What should be" and "What it is"

### step2: Clarify what's unclear
* [ ] From the input, clarify unclear info by AskUserQuestion

### step3: Investigate by "a blank slate" sub agents
* [ ] Launch 2 "a blank slate" sub agents
  * [ ] Hand off the information so far to the agents
  * [ ] Then request them to investigate root reason
* [ ] Aggregate results from the agents

### step4: Reporting
* [ ] Compile the results and report
  * [ ] Report format is below "report format"
* [ ] Out report file to specified path
  * [ ] AskUserQuestion if user did not specify file path

**"report format"**:

```
# Report about {title}
## Point/Conclusion
{Write point/conclusion about investigation}

## Reason
{List: Write reasons why point is valid}

## Example
{List: Write example why reason is appropriate}

## Remarks
{Write remarks if it's required}
```
