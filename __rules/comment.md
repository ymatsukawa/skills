# Rule of comment to source code
## How many
One line.
If any reason, it's permitted 2 lines.

## Content
Write "Why not".
Do not write explicit content.

## Example
OK:
```
# NOTE: In production, SMTP is not used.
# config.smtp_enable = false
```

NG-1: Pointless
```
// NOTE: config.smtp_enable is for SMTP.
// In this product, env is divided to dev/stg/production
// At production, SMTP is not used.
// So SMTP is disabled.
// config.smtp_enable = false
```

NG-2: "Thought process" is written
```ruby
// NOTE: Line of 57 in SPECIFICATION.md, disabled SMTP is requested.
// So, SMTP is disabled in production
// config.smtp_enable = false
```

NG-3: Explicit comment
```
// Declare 3
int num = 3
```