# Format for software feature

```
# Gap Analysis of blog posting
## AsIs
**What it does**
Anonymous user can read blog without authentication.

**Who uses it & when**
* Persona: Anonymous user
* Trigger: Accessing blog
* Expects: Showing blog contents

**Current Behavior - valid/invalid**
* valid
  * Can be shown blog content
  * Cannot be shown blog content if content is hidden mode
* invalid
  * 404 Error when access to non exist blog content

## ToBe
**Changed Behavior - valid/invalid**
* valid
  * (keep) Can be shown blog content
  * (change) **Hidden mode is deleted.**
    * All hidden blog item should be soft deleted.
* invalid
  * (keep) 404 Error when access to non exist blog content

## Problem
There are no specification about soft deletion.
* whether "soft deleted item(s) should be shown to editor"
* who run "first soft deletion". Is that batch? when to do?

## Tasks
* Make resolution about soft deletion with PdM
  * Until 2026/06/2nd week
* Make decision how to soft delete with dev team
  * Until 2026/06/3rd week
```