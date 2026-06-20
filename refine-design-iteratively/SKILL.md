---
name: refine-design-iteratively
description: Refine design iteratively for required intent
---

# Refine design iteratively
## GOAL
Refine target design iteratively until user get satisfied.

## Why
Designing with AI and user is "one round trip" in other word, one way question and just answer.

It's not considered that user's full thought.

In order to input user as human's intention, continue iteration for refinement.

## Prohibition
* `git add` and `git commit`
* Editing exist source code

## Workflow
### step-1: Setup
* [ ] Receive target design sheet or paper
  * **If user did not specify the path, out error like "Cannot process" and stop workflow**
* [ ] Scan the required design.
  * If it's unclear what is design, AskUserQuestion what/how is design.
* [ ] AskUserQuestion that "Whether overwrite or create new design sheet/paper"
  * If "create" is answered, question to user where to output

### step-2: Run iteration
**iteration-1**:
* [ ] Re-scan all around of target design in order to understand the whole

**iteration-2**:
* [ ] By AskUserQuestion, check follows of designing as refinement
  * [ ] Consistency
  * [ ] Tradeoff and alternative
  * [ ] Restrictions that should be placed
  * [ ] And any proposal for refinement

**iteration-3**:
* [ ] Reflect refinement to target doc of the design
* [ ] After reflection, AskUserQuestion that "Refine more or Enough"
  * If "Refine more", go to "iteration-1"
  * If "Enough", stop iteration and workflow is over

**ATTENTION**:
Do not make proposal only "good-side". Designing is almost occupied by "bad-side and pitfall".