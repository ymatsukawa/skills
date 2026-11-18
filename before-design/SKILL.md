---
name: before-design
description: Write a concerns before system designing. Run only when the user explicitly calls this skill.
---

# Before design
## GOAL
Turn out what the "user does not realize" but "you realized" before designing system.

## Why
Most instructions given to an AI amount to "just do it nicely".

But "nicely" does not work. It is either too abstract, or the user does not yet know what should be designed.

To avoid this, you write an concerns about design before decision.

## Goal of skill
Present the concerns about user's thought toward basic designing.

## Prohibition
* `git add`, `git commit`
* Editing files other than the specified one

## Workflow
### step1: Setup
- [ ] Receive from the user what they want to design
  - **If the user gives no concrete instruction, raise an error such as "cannot proceed" and stop the workflow**
- [ ] Understand what is being designed and its target

### step2: Question about obscureness
- [ ] Ask the user with AskUserQuestion about what is unclear to you
  - Repeat AskUserQuestion until any concerns cleared
- [ ] Cover everything the design needs

### step3: Write interview sheet
- [ ] Write the sheet in the "Concerns sheet frame" format
  - [ ] If no output path is given, ask the user with AskUserQuestion where to write it
- [ ] Put it the output path, then it's end

## Concerns sheet frame

### Frame
```markdown
# {Title about design}
## Background
{Write background why all designs are required}

## Scope
**In scope**
- {Write what this design covers}
- ...

**Non-goals**
- {Write what this design does not cover}
- ...

## Concerns summary
| # | Title | Priority | Plan |
|---|---|---|---|
| 1 | {title} | High / Mid / Low | A |
| 2 | ... | ... | ... |

## Concerns-1: {title}
**Priority**: High / Mid / Low
**Impact**: {Write what is affected}

### Target
{Target design or object}

### Why
- {Write reason why target is picked up}
- ...

### What is concerned
**Point of add/mod/del**
- {Write concerns}
- ...

### Risk if ignored
- {Write what happens if this concern is left}
- ...

### Plans to resolve
**Plan-A(Recommended)**:
{Detail of plan-A with code}

**Plan-B**:
{Detail of plan-B with code}

**Why Recommended Plan-A**:
- {Write reason to take plan-A brief in list}
- ...
- ...

**Tradeoff of Plan-A**:
- {Write trade off of plan-A brief in list}
- ...
- ...

**Why not Plan-B**:
- {Write reason to reject plan-B brief in list}
- ...

## Concerns-2: {title}
(Same format as Concerns-1)

# Reference
- [How to fix the framework](http://framework.example.com/fix-to-v10)
```

### Example of output
````markdown
# Authentication for admin console
## Background
The admin console has no login. Anyone on the internal network can open it.

## Scope
**In scope**
- Login and session handling for the admin console

**Non-goals**
- Login for the customer-facing app
- SSO integration

## Concerns summary
| # | Title | Priority | Plan |
|---|---|---|---|
| 1 | Session vs JWT | High | A |

## Concerns-1: Session vs JWT
**Priority**: High
**Impact**: All admin console pages and the API server

### Target
Auth middleware of the API server

### Why
- The console and the API run on the same domain
- Admins need to force logout of a leaked account

### What is concerned
**Point of add**
- No way to revoke a JWT before it expires

### Risk if ignored
- A leaked token stays valid until expiry

### Plans to resolve
**Plan-A(Recommended)**:
Server-side session stored in Redis.
```ts
app.use(session({ store: new RedisStore({ client }), cookie: { httpOnly: true, secure: true } }))
```

**Plan-B**:
Stateless JWT in the Authorization header.
```ts
app.use(jwt({ secret: process.env.JWT_SECRET, algorithms: ["HS256"] }))
```

**Why Recommended Plan-A**:
- Delete the session key to revoke at once
- Same domain, so cookies work without CORS setup

**Tradeoff of Plan-A**:
- Adds Redis as a dependency
- Every request reads Redis

**Why not Plan-B**:
- Revocation needs a deny list, which is a session store in the end

# Reference
- [express-session](https://github.com/expressjs/session)
````
