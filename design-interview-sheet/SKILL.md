---
name: design-interview-sheet
description: Write a design interview sheet through an interview. Run only when the user explicitly calls this skill.
---

# Design interview sheet
## GOAL
Turn out what the user wants to design into an interview sheet, through an interview.

## Why
Most instructions given to an AI amount to "just do it nicely".

But "nicely" does not work. It is either too abstract, or the user does not yet know what should be designed.

To avoid this, write an interview sheet and ask the user questions until "what should be done" is clear.

## Goal of skill
Present the designs that follow from the user's request, split into a recommended one and the alternative.

The workflow ends when the sheet is written. Filling in `### Decision` and acting on it are out of scope: do not wait for the user's checkmarks, do not read them back, do not implement or fix.

## Prohibition
* `git add`, `git commit`
* Editing files other than the specified one

## Workflow
### step1: Setup
- [ ] Receive from the user what they want to design
  - **If the user gives no concrete instruction, raise an error such as "cannot proceed" and stop the workflow**
- [ ] Understand what is being designed and its target

### step2: Question about obscureness
- [ ] Ask the user with AskUserQuestion about what is unclear to you: the background knowledge and the material you need to judge the "design questions"
  - ATTENTION: The questions are not "for the design of step3" but "to remove what you do not know"
- [ ] Cover everything the design needs

### step3: Write interview sheet
- [ ] Write the sheet in the "Interview sheet frame" format
  - [ ] If no output path is given, ask the user with AskUserQuestion where to write it
- [ ] Put it the output path, then it's end

## Interview sheet frame

### Frame
```markdown
# {Title about design}
## Background
{Write background why all designs are required}

## Design-1: {title}
### What will be changed
{description in one line}

### Target
- {Target of design}
- ...

### Why
- {Write reason of design brief in list}
- ...

### Risk of not doing
- {Write risk of not doing brief in list}
- ...

### Plans
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

### Decision
* [ ] Plan-A
* [ ] Plan-B
* [ ] Another way
  * __WRITE_HERE__

## Design-2: {title}
(Same format as Design-1)

# Reference
- [How to fix the framework](http://framework.example.com/fix-to-v10)
```

### Example of output
````markdown
# Make web service more secure
## Background
An outside audit found `/admin` is open to the internet. Only a session cookie check stands in front of it.
"The only team access" must be safer, but not on which layer does the job: nginx, the app, or both.

## Design-1: Access of admin
### What will be changed
Add ratelimit to nginx's config or add restriction to app's logic.

### Target
- HTTP server's config
  - `infra/nginx/conf.d/app.conf`: line 96-100
- Application login logic
  - `src/routes/admin.ts`: line 20-35

### Why
- `/admin` answers any source IP, so a stolen session cookie is enough to get in
- The `isAdmin` check sits inside each handler, so a new route without it is open on the day it ships
- nginx logs the request but not who sent it, so an incident cannot be traced back

### Risk of not doing
- One leaked cookie gives full admin from anywhere
- A forgotten check in a new route is a hole no test catches
- After an incident, no way to answer "who did what"

### Plans
**Plan-A(Recommended)**:
Drop non-office traffic at nginx, and move the admin check to a router-level middleware.

```nginx
# infra/nginx/conf.d/app.conf
set_real_ip_from 10.0.0.0/8;   # only the load balancer may set X-Forwarded-For
real_ip_header   X-Forwarded-For;

location /admin/ {
    allow 10.20.0.0/16;     # office VPN
    allow 203.0.113.10/32;  # bastion
    deny  all;
    proxy_pass http://app_upstream;
}
```

```ts
// src/routes/admin.ts
const admin = Router();
admin.use(requireAdmin);   // every route mounted below inherits the check

admin.get("/users", listUsers);
admin.post("/users/:id/ban", banUser);
```

**Plan-B**:
Leave nginx as is. Do it all in the app: one middleware on `/admin`, plus an MFA re-check every 15 minutes.

```ts
// src/middleware/require-admin.ts
export const requireAdmin: RequestHandler = (req, res, next) => {
  const user = req.session.user;
  if (!user?.isAdmin) return res.sendStatus(404);   // 404, not 403: do not tell a guest the page exists
  if (Date.now() - (user.mfaVerifiedAt ?? 0) > 15 * 60_000) {
    return res.redirect(`/mfa?next=${encodeURIComponent(req.originalUrl)}`);
  }
  next();
};
```

**Why Recommended Plan-A**:
- Outside traffic is cut before it reaches Node, so an app bug under `/admin` cannot be reached from the internet
- The allow list is one block in one file, so a reviewer can read who is let in
- The middleware is the second layer: a route added later is covered without the author remembering

**Tradeoff of Plan-A**:
- Admins cannot work without the VPN, so on-call from a phone stops working
- The CIDR list is one more thing to keep up to date, and it goes stale quietly
- If `set_real_ip_from` and the load balancer disagree, the allow list either lets everyone in or locks everyone out

### Decision
* [ ] Plan-A
* [ ] Plan-B
* [ ] Another way
  * __WRITE_HERE__

## Design-2: Audit log of admin operations
(Same format as Design-1)

# Reference
- [nginx: ngx_http_access_module](http://nginx.org/en/docs/http/ngx_http_access_module.html)
- [Express: Using middleware](https://expressjs.com/en/guide/using-middleware.html)
````

# Requirements in skill

## "Plans"
* Tradeoff of Plan-A
  * Show the "pitfall" behind what makes Plan-A attractive
* Role of Plan-B
  * Serve as the "alternative" for when the tradeoff of Plan-A cannot be accepted
  * Must not exist to "make Plan-A look better"
  * Must not be treated as a "decoy"