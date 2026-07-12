# Avoid guidance of "Data Types"

## Confusing octal literals
A leading `0` (or `0o`) makes a literal octal, which is easy to misread as decimal. Be explicit and add comments.

NG:
```go
total := 200 + 010 // 010 is octal 8, not 10 → total is 208, not 210
```

OK:
```go
total := 200 + 0o10 // 0o prefix clearly signals octal (value 8)
```

## Neglecting integer overflows
Integer arithmetic silently wraps around on overflow. Detect it explicitly before it happens.

NG:
```go
func NextSeq(seq int32) int32 {
	return seq + 1 // wraps to MinInt32 when seq == MaxInt32
}
```

OK:
```go
func NextSeq(seq int32) int32 {
	if seq == math.MaxInt32 { // check before incrementing
		panic("int32 overflow")
	}
	return seq + 1
}
```

## Not understanding floating-point arithmetic
float32/float64 are approximations; results depend on operand magnitude and order. Group operations of similar magnitude.

NG:
```go
acc := 50_000.
for i := 0; i < n; i++ { // adding tiny values to a large one loses precision
	acc += 1.0002
}
```

OK:
```go
acc := 0.
for i := 0; i < n; i++ { // accumulate small values first
	acc += 1.0002
}
return acc + 50_000.
```

## Confusing slice length and capacity
`len` is the number of accessible elements; `cap` is the backing array size. `append` only grows length until capacity is exceeded, then reallocates.

NG:
```go
nums := make([]int, 3, 6)
_ = nums[4] // panic: index out of range (within cap but beyond len)
```

OK:
```go
nums := make([]int, 3, 6) // len=3, cap=6
nums = append(nums, 7)    // len=4, still cap=6, no reallocation
```

## Inefficient slice initialization
When the final size is known, preset capacity (or length) to avoid repeated reallocation and copying during `append`.

NG:
```go
reports := make([]Report, 0) // grows and reallocates repeatedly
for _, task := range tasks {
	reports = append(reports, taskToReport(task))
}
```

OK-1:
```go
reports := make([]Report, 0, len(tasks)) // preallocate capacity, then append
for _, task := range tasks {
	reports = append(reports, taskToReport(task))
}
```

OK-2:
```go
reports := make([]Report, len(tasks)) // preset length, assign by index
for i, task := range tasks {
	reports[i] = taskToReport(task)
}
```

## Confusing nil and empty slices
A nil slice and an empty slice both have len 0, but differ: nil marshals to `null`, empty to `[]`. Favor nil declaration unless an empty-but-non-nil semantic is needed.

NG:
```go
ratios := make([]float64, 0) // non-nil empty slice; allocates, marshals to []
```

OK:
```go
var ratios []float64 // nil slice, no allocation; marshals to null
```

## Improperly checking if a slice is empty
A non-nil empty slice is not nil, so `!= nil` can be wrong. Use `len`.

NG:
```go
if pending != nil { // true even for an empty (non-nil) slice
	process(pending)
}
```

OK:
```go
if len(pending) != 0 { // works for nil and empty slices alike
	process(pending)
}
```

## Not making slice copies correctly
`copy` copies only min(len(dst), len(src)) elements, so a nil/zero-length dst copies nothing. Size dst first.

NG:
```go
var to []int
copy(to, from) // copies 0 elements; to stays empty
```

OK:
```go
to := make([]int, len(from)) // size the destination to the source's length
copy(to, from)
```

## Unexpected side effects using append
`append` on a sub-slice that shares a backing array can overwrite the original's elements. Copy or use a full slice expression to isolate.

NG:
```go
nums := []int{4, 5, 6}
consume(nums[:2]) // append inside consume writes into nums[2], mutating nums
```

OK-1:
```go
numsCopy := make([]int, 2) // independent copy
copy(numsCopy, nums)
consume(numsCopy)
```

OK-2:
```go
consume(nums[:2:2]) // full slice expr caps capacity, forcing a new backing array on append
```

## Slices and memory leaks
A slice keeps its whole backing array alive, and pointer/struct elements left in the tail stay referenced. Copy out, or nil unused elements.

NG:
```go
func extractHeader(packet []byte) []byte {
	return packet[:8] // retains the full 1MB backing array
}
```

OK-1:
```go
func extractHeaderWithCopy(packet []byte) []byte {
	header := make([]byte, 8) // copy frees the original array
	copy(header, packet)
	return header
}
```

OK-2:
```go
func trimToFirstTwoMarkNil(tasks []Task) []Task {
	for i := 2; i < len(tasks); i++ {
		tasks[i].payload = nil // release pointer-held memory in the tail
	}
	return tasks[:2]
}
```

## Inefficient map initialization
Without a size hint, a growing map rehashes and reallocates buckets repeatedly. Provide the expected size.

NG:
```go
seen := make(map[int]struct{}) // rehashes as it grows
for i := 0; i < n; i++ {
	seen[i] = struct{}{}
}
```

OK:
```go
seen := make(map[int]struct{}, n) // preallocate buckets for n entries
for i := 0; i < n; i++ {
	seen[i] = struct{}{}
}
```

## Maps and memory leaks
A map's bucket count only grows; deleting keys does not shrink it, so memory stays high. Recreate the map or store pointers for large values.

NG:
```go
cache := make(map[int][128]byte)
for i := 0; i < n; i++ { cache[i] = newBlob() }
for i := 0; i < n; i++ { delete(cache, i) } // buckets remain allocated
```

OK:
```go
cache := make(map[int]*[128]byte) // store pointers so deleted values can be GC'd
for i := 0; i < n; i++ { cache[i] = newBlobPtr() }
// or rebuild the map periodically to reclaim bucket memory:
old := cache
cache = make(map[int]*[128]byte, len(old))
for k, v := range old { cache[k] = v }
```

## Comparing values incorrectly
`==` works only on comparable types and compares fields shallowly; a slice/map/func field makes a struct non-comparable. Use `reflect.DeepEqual` or a custom method.

NG:
```go
order1 == order2 // compile error: struct contains a slice field
```

OK-1:
```go
reflect.DeepEqual(order1, order2) // deep comparison (slower, reflection-based)
```

OK-2:
```go
func (a order) equal(b order) bool { // explicit, fast comparison
	// compare fields and slice elements manually
}
```
