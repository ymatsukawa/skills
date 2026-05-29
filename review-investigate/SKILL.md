---
name: review-investigate
description: Review investigated result by reviewer of a blank state
---

# Review investigate
## GOAL
Review investigated result by reviewer of a blank state.

## Why
When investigation finished, the result is assumed "correct" because of scotoma.

In order to avoid the unwitting situation, this skill reviews and find the dark point; scotoma.

## Definition
**scotoma**:
Blind spot or patch in vision where simply cannot see because of the bias that "I must know this already" or indifference of mind.

## Prohibition
* Editing source code.
* `git add` and `git commit`

## Workflow
### step1: Receive result of investigation
* [ ] Receive input from user
  * **If no input, out error like "Cannot process because no information", and stop workflow.**

### step2: Understand report
* [ ] At first, understand "What happened and what is problem"
* [ ] Then watch "What is the reason(s) of the problem"
* [ ] If any unclear items, AskUserQuestion

### step3: Call supporter to investigate again
* [ ] Launch subagent "fresh researcher" as "a blank state" agent
* [ ] Pass information so far
* [ ] As the agent, review the investigation
  * [ ] Is "conclusion/point" correct? why?
  * [ ] Is "reason" valid? why?
* [ ] Receive report from the agent

### step4: Reporting
* [ ] Report with "review-report-format"
* [ ] Then out to specified path
  * If user did not point path, AskUserQuestion where to out

**"review-report-format"**:

```
# Review of {Title}
## Conclusion/Point
**Approve | Conditionally Approve | Reject**

## Reasons
{List: Write down why getting the conclusion, especially "Conditionally Approve" and "Reject"}

## Example
{List: Write down reinforcements of reasons}

## Remarks
{Write remarks if exist}
```
