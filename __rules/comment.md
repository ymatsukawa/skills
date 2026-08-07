# Rule of comment to source code
## How many
One line.
If any reason, it's permitted 2 lines.

## Content
Write "Why not" for intention.

Do not write explicit content.

This is because of quote from Martin Fowler.

> Any fool can write code that a computer can understand.
> Good programmers write code that humans can understand.

## Examples
OK:
```
// In production, SMTP is not used.
// config.smtp_enable = false
```

NG-1: Pointless
```
// config.smtp_enable is for SMTP.
// In this product, env is divided to dev/stg/production
// At production, SMTP is not used.
// So SMTP is disabled.
// config.smtp_enable = false
```

NG-2: "Thought process of LLM" is written
```
// Line of 57 in SPECIFICATION.md, disabled SMTP is requested.
// So, SMTP is disabled in production
// config.smtp_enable = false
```

NG-3: Explicit comment is written
```
// Declare 3
int num = 3
```

NG-4: Explaining spec of programming language
```go
s := "red,green,blue"
// split string to 3 elements of []string
arr := strings.Split(s, ",")
```