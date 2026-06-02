---
name: acceptance-scenario-creation
description: Create acceptance test scenario with specific standard.
---

# Acceptance scenario
## GOAL
Creating acceptance test scenario with Gherkin style.

## Why
Many developers input prompt just "create test".
But this will not satisfy them, because there is no standard.
Is it about unit? joint? acceptance? It's not unclear.

This skill focuses on "acceptance tests" and built the cases.

## Definition
**acceptance tests**:
Testing behaviors by viewpoint of "Business" and "User" not by "Developer".
In other word, it's called UAT; User Acceptance Test.

**worthless-testing**:
Acceptance tests that produces no value for business and user of the service.

In another word, "tests for developer".

For example:
* Checking API response header
* Checking value of array
* Checking error "process" of server

These are not worthless for the business and user.

**scoped out test**:
Thinking about belows test pattern. They are in scope of "system test"
* Load test
* Long running test
* Security test
* Performance test
* Memory leak test
* Migration test
* Concurrency/Parallelism test

## Workflow
### step-1: Setup
* [ ] Receive test target(s) of source code or design that can be understood "system behavior"
  * **If not specified target(s), out error like "Cannot process. Pass target source or design." and stop workflow**
* [ ] Scan target and understand content.
  * If any unclearness, AskUserQuestion until no vague.

### step-2: Plan for acceptance testing
* [ ] With "standard of acceptance test", create plan of acceptance test
* [ ] After plan, format them as "Gherkin style"

**"standard of acceptance test"**:
* Avoid "worthless-testing"
* Scope should not include "scoped out test"

**"Gherkin style**:
```markdown
# Valid Scenarios
## Feature: Post blog as member
* **Given**: Login as "user@example.com" with "password"
* **When**: Visit to "Blog create" page
* **And**: Input title as "Hello world"
* **And**: Input content as "This is first posting..."
* **And**: Click "Post" button
* **Then**: "Success post" page is shown
* **And**: The page have "share link" button
* **When**: Click "share link" button and paste it to memo
* **Then**: Share link is shown as "https://blog.example.com/..."

## Feature: Edit blog as member
...

# Invalid Scenarios
## Feature: Ban member cannot login
* **Given**: "ban@example.com" is banned by admin
* ...
```

### step-3: Out file
* [ ] Out file to specific path
  * If user does not pass output path, AskUserQuestion where to out