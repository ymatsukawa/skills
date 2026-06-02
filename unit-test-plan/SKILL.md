---
name: unit-test-plan
description: Create unit test plan with specific standard.
---

# Unit test plan
## GOAL
Creating unit test plan with specific standard.

## Why
Many developers input prompt just "create unit test".
But this will not satisfy them, because there are not standards.

In order to clear "what to do", this skill clears standards.

## Definition
**unit test**:
Verifying "the module/class" or "the method" is valid or invalid.

remarks:
* integration test: Verifying communication result with "many modules"
* acceptance test: Verifying user's expectation

**standard of unit test**:
Read: `references/unit-test-standard.md`

**over-testing**:
Tests that produces no value for developers and users of service.

In another word, "test for test".

For example, when checking element of array integer which is strictly typed, verifying element type is not hash or custom class.


**out of test**:
About belows test pattern. They are not unit test scope.
* Load test: Needs mass data; unit tests should be light.
* Long running test: Takes long; unit tests should be short.
* Security test: Needs security tool; unit tests check functionality.
* Performance test: Below the functionality layer.
* Memory leak test: Below the functionality layer.
* Migration test: Needs rehearsal data; unit tests use fake data.
* Concurrency/Parallelism test: Results vary; unit tests must be 100% reproducible.
* Contract test: Depends on other services; out of "unit".

## Workflow
### step-1: Setup
* [ ] Receive test target(s) source code, directory or test itself.
  * **If not specified target(s), out error "Cannot plan. Pass target source file, directory or test file." and stop workflow**
* [ ] Scan target(s) file(s)

### step-2: plan by standard
* [ ] With "standard of unit test", create plan how to unit test
  * Remember, **AVOID 'over-testing' and 'out of testing'**

### step-3: pass plan to user
* [ ] Pass plan result to user
  * If not specified format, use below "unit test plan format"
  * If user required as file, out with markdown and AskUserQuestion where to.

**unit test plan format**:
```
# Planned unit test design

## Black-box test
{planned test details}

## White-box test
{planned test details}
```