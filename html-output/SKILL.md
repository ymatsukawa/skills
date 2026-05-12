---
name: html-output
description: Output readable html from text, markdown or other format.
---

# Html output

## 2 methods of skill
* Just make "static" html.
* Follow references, don't arrange.

## Workflow
### Requirements
* Receive target file path
  * If no file path, out error "Specify target", and stop workflow

### 1st step
* [ ] Determine what type of the content. Skill cover follow types
  * `analysis`: AsIs-ToBe, system flow
  * `design`: mockup
  * `diff`: diff of code
  * `plan`: task planning
  * `proposal`: SPIN proposal
  * `report`: reporting bug, incident
* [ ] When type matched nothing, out error "Your requirement is not covered" and stop workflow

### 2nd step
* [ ] Generate new html following design of referenced html
  * `analysis`: `references/analysis-*.html`
  * `design`: `references/design-*.html`
  * `diff`: `references/diff-*.html`
  * `plan`: `references/plan-*.html`
  * `proposal`: `references/proposal-*.html`
  * `report`: `references/report-*.html`
* [ ] Output html to specified path by user
  * If no target path, AskUserQuestion where to output

## Formatting
* UTF-8

## Prohibition
* Using javascript
* Editing original input
