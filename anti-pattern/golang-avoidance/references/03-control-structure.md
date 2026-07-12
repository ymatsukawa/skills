# Avoid guidance of "Control Structures"

## Ignoring that range-loop elements are copies
The range value variable is a copy of the element, so mutating it does not affect the source slice. Index the slice directly (or use a pointer slice) to mutate in place.

NG:
```go
// 'w' is a copy; original slice is unchanged
for _, w := range wallets {
	w.points += 500
}
```

OK:
```go
// index into the slice to mutate the real element
for i := range wallets {
	wallets[i].points += 500
}
```

## Range expression is evaluated only once
The range expression is evaluated once at loop start (and arrays are copied), so changes made during iteration to the original are not seen. Use a classic index loop or `range &a` if updates must be visible.

NG:
```go
// ranges over a copy of the array; v is the original 7, not 99
arr := [3]int{5, 6, 7}
for i, v := range arr {
	arr[2] = 99
	if i == 2 { fmt.Println(v) } // 7
}
```

OK-1:
```go
// index loop reads the live array
for i := range arr {
	arr[2] = 99
	if i == 2 { fmt.Println(arr[2]) } // 99
}
```

OK-2:
```go
// ranging over a pointer avoids the copy
for i, v := range &arr {
	arr[2] = 99
	if i == 2 { fmt.Println(v) } // 99
}
```

## Using pointers to a range-loop variable
The range variable is reused across iterations, so taking its address stores the same pointer every time (all pointing to the last value). Copy into a local var or address the indexed element instead.

NG:
```go
// &article is the same address each iteration
for _, article := range articles {
	c.byID[article.ID] = &article
}
```

OK-1:
```go
// local copy gives each entry its own address
for _, article := range articles {
	a := article
	c.byID[a.ID] = &a
}
```

OK-2:
```go
// point directly into the slice element
for i := range articles {
	article := &articles[i]
	c.byID[article.ID] = article
}
```

## Making wrong assumptions about map iteration
Map iteration order is unspecified, and entries added during iteration may or may not be produced. Don't add to the map you are ranging over; build a separate map.

NG:
```go
// adding keys while ranging gives nondeterministic results
for k, v := range flags {
	if v { flags[k+100] = true }
}
```

OK:
```go
// write into a copy, not the map being iterated
next := cloneMap(flags)
for k, v := range flags {
	next[k] = v
	if v { next[k+100] = true }
}
```

## Ignoring how break/continue work with switch/select
A bare `break` inside a `switch`/`select` breaks that statement, not the enclosing loop. Use a labeled `break` to exit the loop.

NG:
```go
// break exits the select, not the for loop (loops forever)
for {
	select {
	case <-ctx.Done():
		break
	}
}
```

OK:
```go
// labeled break exits the loop
poll:
	for {
		select {
		case <-ctx.Done():
			break poll
		}
	}
```

## Using defer inside a loop
`defer` runs at function return, so deferring in a loop keeps every resource open until the function ends. Wrap the body in a function so defers fire per iteration.

NG:
```go
// files stay open until the enclosing function returns
for name := range paths {
	f, _ := os.Open(name)
	defer f.Close()
}
```

OK-1:
```go
// extract a function so defer runs each call
for name := range paths {
	if err := loadConfig(name); err != nil { return err }
}
func loadConfig(name string) error {
	f, _ := os.Open(name)
	defer f.Close()
	return nil
}
```

OK-2:
```go
// inline closure executed per iteration
for name := range paths {
	err := func() error {
		f, _ := os.Open(name)
		defer f.Close()
		return nil
	}()
	if err != nil { return err }
}
```
