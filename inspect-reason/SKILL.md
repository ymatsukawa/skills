---
name: inspect-reason
description: Inspect reason of problem by zero-based inspection. This skill should be launched only when called explicitly.
---

# Investigate reason
## GOAL
Survey reason of problem.

## Why
When staying on the project for a long time, developers start making biased assumptions.

In order to avoid the assumptions, this skill begins "zero-based" inspection.

## Prohibition
* Editing source files.
* `git add` and `git commit`

## No need
* Plans with how to fix
  * You just focus on inspection

## Workflow
### step1: Receive what user want to resolve
* [ ] Receive request from user what to resolve
  * **If user does not specify the input, out error like "Cannot start skill because any info about problem", and stop workflow.**

**Strongly recommended**:
Sheet or paper has "What should be" and "What it is"

### step2: Clarify what's unclear
* [ ] From the input, clarify unclear info by AskUserQuestion

### step3: Investigate by "a blank slate" sub agents
* [ ] Launch 2 "a blank slate" sub agents
  * [ ] Hand off the information to the agents
  * [ ] Then request them to inspect root cause, downstream effect and causal chain
* [ ] Aggregate results from the agents

**Why 2 agents?**:
Only 1 agent may mistakes or drops cause.

In order to "double-check", 2 agents are required.

### step4: Reporting
* [ ] Compile the results and report
  * [ ] Report format is below "report format"
* [ ] Out report file to specified path
  * [ ] AskUserQuestion if user did not specify file path

**"report format"**:

```markdown
# {title}
## Reason-1: {Title}
* Type: Root cause

### Summary of reasons
- {List: Write reasons}
- ...
- ...

### Detailed reason
{Write target source code}

- {Explain what is reason}
- ...
- ...

### Target
- `/path/to/code.ext`
- `/path/to/code.ext`

## Reason-2: {Title}
* Type: downstream effects
(same format as Reason-1)

## Extra-1: {Title}
* Type: causal chain; you should fix at the same time
(same format as Reason-1)
```

**Example of report format**:
````markdown
# Why backend gets slow irregularly

## Reason-1: N+1 query in order listing for large tenants
* Type: Root cause

### Summary of reasons
- `ListOrders` loads orders in one query, then runs one `SELECT` per order to fetch its customer
- Query count equals order count, so a tenant with tens of thousands of orders takes seconds while small tenants finish in milliseconds
- Slowness looks irregular because it depends on which tenant hits the endpoint

### Detailed reason
```go
// internal/order/repository.go
rows, _ := r.db.QueryContext(ctx, `SELECT id, customer_id, total FROM orders WHERE tenant_id = $1`, tenantID)
for rows.Next() {
    var o Order
    rows.Scan(&o.ID, &o.CustomerID, &o.Total)
    // one round trip per order
    r.db.QueryRowContext(ctx, `SELECT name FROM customers WHERE id = $1`, o.CustomerID).Scan(&o.CustomerName)
    orders = append(orders, o)
}
```

- The customer lookup runs inside the row loop, so 40,000 orders issue 40,001 queries.
- Each round trip costs about 1 ms on the current network, which alone accounts for the 40 s response seen in that tenant's logs.

### Target
- `internal/order/repository.go`
- `internal/order/handler.go`

## Reason-2: Connection pool exhaustion blocks unrelated endpoints
* Type: downstream effects

### Summary of reasons
- Each per-order query borrows a connection from a pool capped at 20
- While one large-tenant request runs, other requests wait for a free connection
- Endpoints that never touch orders also slow down, which hides the true origin

### Detailed reason
```go
// internal/db/pool.go
db.SetMaxOpenConns(20)
db.SetMaxIdleConns(20)
db.SetConnMaxLifetime(30 * time.Minute)
```

- The pool is fixed and shared by all handlers.
- Two concurrent large-tenant requests keep it near saturation for tens of seconds, so `/health` and `/me` queue behind them and show latency spikes with no code path in common.

### Target
- `internal/db/pool.go`
- `internal/health/handler.go`

## Extra-1: Repository ignores request context, so slow queries never cancel
* Type: causal chain; you should fix at the same time

### Summary of reasons
- Handler passes `context.Background()` to the repository instead of the request context
- When a client gives up and disconnects, the query loop keeps running to completion
- Abandoned requests keep holding connections, which prolongs Reason-2

### Detailed reason
```go
// internal/order/handler.go
func (h *Handler) ListOrders(w http.ResponseWriter, req *http.Request) {
    tenantID := tenantFrom(req)
    orders, err := h.repo.ListByTenant(context.Background(), tenantID) // should be req.Context()
    ...
}
```

- Fixing the N+1 alone leaves a path where any future slow query outlives its request.
- Passing `req.Context()` lets cancellation propagate, so the pool recovers as soon as clients time out.

### Target
- `internal/order/handler.go`
- `internal/order/repository.go`
````