---
name: create-release-note-internal-publication
description: Create an internal release note from a git revision range. Use when the user asks for a release note between two branches or commits for internal announcement.
---

# Create release note (internal publication)
## Goal
Create a release note for internal announcement.
Readers must understand at a glance how the release affects the product.

## Why
Writing release notes by hand takes time. This skill automates it.

## Prohibition
* `git add` and `git commit`
* Editing existing files

## Usage
* `/create-release-note-internal-publication from main to feature/add-realtime-edit`
* `/create-release-note-internal-publication from 03e8a55 to 332cdd5`

## Workflow
### step1: Setup
* [ ] Check whether user installed `gh` command
  * If it's not installed, stop workflow and request to install
* [ ] Receive "from" and "to" (branch or revision) from user.
  * If either is missing, output "Cannot process: 'from' and 'to' are required", show Usage, and stop.
* [ ] Get changes:
  * Commits: `git log <from>..<to> --oneline`
  * Diff: `git diff <from>...<to>` (three-dot, to exclude changes only on <from>)
* [ ] Get PR links:
  * `git log <from>..<to> --merges --oneline` to find PR numbers
  * `gh pr view <number> --json title,url` for each

### step2: Grouping
Group each change into one of:
* Feature: visible to users
* Bugfix: fixes broken behavior
* Refactor: internal change, not visible to users
* Other: docs, CI, dependency updates. Put under "No impact on users".

### step3: Create note
* [ ] Fill the template in `references/for-non-engineer.md`.
  * [ ] If user requested in Japanese use `references/for-non-engineer-ja.md`
* [ ] Output to the specified path. If not specified, output to `./release-note-{from}-{to}.md`.
