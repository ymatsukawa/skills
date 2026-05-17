---
name: gap-analysis
description: Analyze gap between AsIs and ToBe, then find problem and tasks.
---

# Gap Analysis
## Goal
Find gaps by comparing "AsIs" and "ToBe".

## Definitions
**AsIs**:
What it is in current state.
Ignore what should be in future.

**ToBe**:
What should be in future.
Ignore what it is in current state.

**gap**:
The space between AsIs and ToBe.
In other words, this is "problem".

**task(s)**:
When task(s) is finished, problem is solved.
In other words, ToBe is achieved.

## Prohibition
* Read files and directories that is pointed by `.gitignore`
* Editing existing files

## Workflow
There are mainly two steps.
* 1st step: Understand AsIs and ToBe
* 2nd step: Analyze gap
* 3rd step: Output report

### 1st step - Understand AsIs and ToBe
* [ ] Receive sheet that describes "AsIs and ToBe"
  * **If no such sheet, error "Specify AsIs and ToBe" and stop all workflow.**
* [ ] You can output memory under `/tmp` for after step

### 2nd step - Analyze gap
* [ ] Think the gap between AsIs and ToBe
  * [ ] **To clarify gap and reconfirm AsIs and ToBe, use AskUserQuestion.**
* [ ] Organize the gap
* [ ] Think what is task(s)

### 3rd step - Output report
* [ ] Output analyzed info as markdown
  * Format follows "Output format"
  * When user does not specify path, AskUserQuestion where to out

### (optional) 4th step - Reinforce items
* [ ] Ask user whether add belows by AskUserQuestion
  * Prohibition
  * Decisions(Resolved and Unresolved)
  * Considerations

If answer is "add":
* [ ] Insert sections after "Problem" before "Task"
  * Read format `references/reinforcement.md`

**Output format**:
Decide based on following cases.

* (1) Software feature
  * Read: `references/software-feature-case.md`
* (2) General case
  * If none of the above applies
  * Read: `references/general-case.md`
