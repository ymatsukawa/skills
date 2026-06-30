---
name: characterization-test-plan
description: Plan characterization testing for before refactoring
---

# Characterization test plan
## GOAL
Create characterization testing planning.

## Why
In refactoring, it's required "original spec and behavior is not broken".

The request is satisfied by testing, but almost all of projects do not have tests.

In order to fill the request, this skill plans characterization test.

## Definition
**Characterization test**:
Test that actual behavior of an existing piece of software.

This test protects existing behavior against unintended changes.

Also known as "Golden master test".

**over-testing**:
Tests that produces no value for developers and users of service.

In another word, "test for test".

For example, checking element of array integer which is strictly typed.

## Workflow
### step1: Setup
* [ ] Receive target for testing
  * [ ] **If user does not specify target, out error like "Cannot progress" and stop workflow**

**Important**:
This step is just for "receiving target". Scanning and another is later.

### step2: Get ready to plan
* [ ] Scan the received target and check follows
  * [ ] Inputs
  * [ ] Outputs
  * [ ] Dependencies
* [ ] With the attributes, understand **"How behaves actually"**

### step3: Plan
* [ ] Cover all behavior patterns from step-2's understanding
  * [ ] Input is real, dependencies are mock or double
  * [ ] Avoid "over-testing" planning, just cover characterization test

### step4: Report
* [ ] Output report to file
  * [ ] Format is on "Characterization test format"
  * [ ] If user does not specify path, AskUserQuestion where to out

**"Characterization test format"**:

In report, output and behavior is same.

```markdown
# {title}
## Target
{target code}

## Valid test cases
### Returning specified user
**Input**:
* email: `test@example.com`
* user_id: `null`

**Behavior**:
* Returns specified user by test@example.com

### {case name}
...

## Invalid test cases
### Thrown error
**Input**:
* email: null
* user_id: null

**Behavior**:
* Throw error as NotFound

### {case name}
...

## Strategy of mocking dependency 
{Detail of strategy}
```
