---
name: handoff-outlier
description: Capture and write out outlier for handoff
---

# handoff-outlier
## GOAL
Capture and write out "implicit habit" and "tacit knowledge" for handout.

## Why
In general, developer set complex design out of convention, because of tight time limit.
And the design becomes "implicit habit" and "tacit knowledge".
(This skill call them as "outlier" or "camouflage".)

When in handoff, poison of the design suffers newcomer.
In order to decrease the pain as possible, this skill captures and explicitly writes out.

## Definition
**outlier**:
* "implicit habit" and "tacit knowledge"

**standard of outlier**:
* Newcomers cannot understand in 10 minutes and hard to explain.
* FAQs are required for senior workers.

## Prohibition
* Editing files or directories out of skills

## Workflow
### Step 1: Setup
* [ ] Receive target where to search, file? directory? around repository?
  * **If target is not passed, out error "Handoff target is not specified." and stop workflow"**

### Step 2: Scan "main convention"
* [ ] Scan 2 groups
  * [ ] "main convention"
    * Framework?
      * Django?
      * Next.js?
      * Simple React?
      * Then, what's the version?
    * IaC?
      * Terraform?
      * Ansible?
      * Then, what's the version?
    * etc.
  * [ ] Also understand depending "external module"
    * Library?
    * External service?
    * etc.
* [ ] After scan, memory them for after workflow

### Step 3: Find "outlier"
* [ ] Find out outlier
  * Standard is written in "Standard of 'outlier'"
* [ ] Then group them to categories

### Step 4: Write out outlier
* [ ] Write outlier ot specified path
  * Format is below "Output format"
  * If not received path, AskUserQuestion where to out

**Output format**:
```
# Handout: outliers in {target}
## Convention
* Framework: {name @ version}
* IaC: {name @ version}
* ...(another items)
* Key external modules:
  * {list}
  * ...

Anything below is an outlier.

## 1. Naming & Conventions

### {short title}
* **What**: {one-sentence description of the deviation}
* **Where**: `{file or glob path}`
* **Why**: {historical reason or constraint}
* **How to apply**: {what you should do when touching this}

### {short title}
...

#### FAQ
* **Q1**: {question get asked repeatedly}
  * **A**: {answer, with `{path}` reference if needed}
* **Q2**: ...
  * **A**: ...

## 2. Domain Logic Vices

## 3. Error Handling
(same structure as section 1; omit if no outliers — write "No outliers found.")

## 4. Testing

## 5. Build & Deploy

## 6. Configuration & Secrets

## 7. External Dependencies

## Cross-cutting notes (optional)
- {anything that spans multiple categories, kept short}
```
