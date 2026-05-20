---
name: impact-analysis
description: Investigate impact about cascading effects of target
---

# Impact analysis
## GOAL
To avoid incident and accident, investigate impact about cascading effects of target.

## Why
There is bias in an overly familiar environment for developers.
So, overlooking becomes normal in investigating.

In order to stay away from the issue, this skill supports to capture fact of impact.

## Prohibition
* Editing files and directories

## Workflow
### step-1: Setup
* [ ] Receive info whats the analysis point from user. Request sheet or paper is enough.
  * **If not passed any, out error "Cannot analyze. Pass the sheet or paper what to do" and stop workflow.**
* [ ] Understand what to investigate

### step-2: Request to "newcomer"
* [ ] Launch **2 sub-agents "newcomer"** who are clean and have no knowledge about target.
* [ ] Pass them information from user
* [ ] Request sub-agents belows
  * "If target had been changed by user request..."
  * "What will be happened in worst case?"
  * "Why the case will be happened?"
  * "Who will be involved?"
  * "What will be the risk level?"
  * "Will it be manageable? Yes or no, why?"

**Prohibition**:
* Each sub-agents should not talk and share progress and result information
  * This is because to dodge "new bias"

### step-3: work of "newcomer"
* [ ] Run investigating by 2 "newcomer" with request of step-2

### step-4: Report by "newcomer"
* [ ] On finished the step-3, gather and compile findings from sub-agents.
  * [ ] After gathering information, exit the agent
* [ ] Report with "format of impact-analysis report"
  * If user does not specify path, AskUserQuestion where to out

**format of impact-analysis**:
```
# Impact analysis of {title}
## Target
{Point target}

## Effects
### {short title}
**Risk**:
high/middle/low

**Summary**:
{description in brief}

**Details**:
{Write detail of impact}

**Involved**:
{list: Write who are involved}

**Manageable**:
yes/no
{Write reason}

### {short title}
...
```