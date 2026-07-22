---
name: create-release-note-internal-publication
description: Create release note of target repository for internal publication.
---

# Create release note
## Goal
Create release note for showcase of internal publication.

## Why
Many developers thought that "creating release note is dull and want to save time".

Because of it's not for "user" but for company's announce.

In order to be more time effort, this skill achieves automating the task as possible.

## Prohibition
* `git add` and `git commit`
* Editing exist files

## Usage
* case1: `/create-release-note-internal-publication from main to feature/add-realtime-edit`
* case2: `/create-release-note-internal-publication from 03e8a55 to 332cdd5`
* case3: `/create-release-note-internal-publication from main..3332cdd5`

## Workflow
### step1: Setup
* [ ] Receive revision or branch about "from" and "to" from user.
  * **If no input or lack one onf them, out error "Cannot process" and stop workflow.w**
  * Show "usage" to user.
* [ ] Switch model to `Opus` and it's recommended higher version.
  * For correct switch, AskUserQuestion that which to switch
* [ ] Get diff of `from..to` and understand diff content

### step2: Grouping
* [ ] Group modifications
  * Feature: Impact to external; reacts to user
  * Refactor: Impact to internal; not react to user
  * Bugfix: Literally bugfix

### step3: Create note
* [ ] Use template from `references/for-non-engineer.md`
* [ ] After all, create release note
* [ ] If output path is not specified, AskUserQuestion where to out.