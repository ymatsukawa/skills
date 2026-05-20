# Tidy First

## Philosophy
This is **NOT big refactoring** but it's elemental cleanup that **makes the next change easier**.

## Methods of tidy first
1. Delete Dead Code
2. Guarding
3. Move Declaration and Initialization Together
4. Explaining Variables
5. Explaining Constants
6. Normalize Symmetry
7. Chunk Consolidation
8. Extract Helper
9. Explicit Parameters
10. Chunk Statements
11. Reading Order
12. Cohesion Order
13. Explaining Comments

## 1. Delete Dead Code
Remove code that never executes — but verify it is truly unused (reflection, dynamic dispatch) first.

```
NG:
function process(data)
  unused_value = calculate()  // never referenced
  return transform(data)

OK:
function process(data)
  return transform(data)
```

## 2. Guarding
Guard conditions early so preconditions are explicit and the main flow stays unnested.

```
NG:
if (condition)
  if (nested_condition)
    return result
  main_process()
if (another_condition)
  return other_result
more_process()

OK:
if (not condition) return
if (not nested_condition) return
if (not another_condition) return

main_process()
```

## 3. Move Declaration and Initialization Together
Keep a variable's declaration close to its initialization and usage — scattered declarations obscure intent.

```
NG:
function process()
  int a
  some_code_not_using_a()
  a = value

  int b
  more_code_using_a_not_b()
  b = calculate(a)
  code_using_b()

OK:
function process()
  some_code_not_using_a()

  int a = value
  int b = calculate(a)
  code_using_b()
```

## 4. Explaining Variables
Extract complex expressions into well-named variables to reveal intent at a glance.

```
NG:
return new Point(
  (x1 + x2) / 2 + offset * cos(angle),
  (y1 + y2) / 2 + offset * sin(angle)
)

OK:
center_x = (x1 + x2) / 2 + offset * cos(angle)
center_y = (y1 + y2) / 2 + offset * sin(angle)

return new Point(center_x, center_y)
```

## 5. Explaining Constants
Replace magic numbers with descriptive symbolic constants — avoid names so generic they lose meaning.

```
NG:
if user.status == 1

NG:
STATUS = 1
if user.status == STATUS

OK:
ACTIVE_STATUS = 1
if user.status == ACTIVE_STATUS
```

## 6. Normalize Symmetry
Use a consistent approach for identical operations — inconsistency implies different intent when none exists.

```
NG:
function_a()
  return foo != nil ? foo : foo = initialize()

function_b()
  foo = foo != nil ? foo : initialize()
  return foo

OK:
function_a()
  if foo is nil
    foo = initialize()
  return foo

function_b()
  if foo is nil
    foo = initialize()
  return foo
```

## 7. Chunk Consolidation
Over-fragmented code hides the big picture — consolidate related code first to see the whole, then re-extract with meaningful abstractions.

Signs you need consolidation: long repeated parameter lists, repeated code (especially conditionals), unclear helper names, shared mutable data structures.

```
NG:
function calculate_total(items)
  subtotal = calculate_subtotal(items)
  tax = calculate_tax(subtotal)
  discount = calculate_discount(subtotal)
  shipping = calculate_shipping(subtotal, discount)
  return apply_final_calculation(subtotal, tax, discount, shipping)

function calculate_subtotal(items)
  sum = 0
  for item in items
    sum += get_item_price(item)
  return sum

function get_item_price(item)
  return item.price * item.quantity

OK-Step1: Consolidate to understand the whole
function calculate_total(items)
  subtotal = 0
  for item in items
    subtotal += item.price * item.quantity

  tax = subtotal * 0.08

  discount = 0
  if subtotal > 10000
    discount = subtotal * 0.1

  shipping = (subtotal - discount > 5000) ? 0 : 500

  return subtotal + tax - discount + shipping

OK-Step2: Re-extract with meaningful units
function calculate_total(items)
  subtotal = sum(item.price * item.quantity for item in items)
  tax = subtotal * TAX_RATE
  discount = get_volume_discount(subtotal)
  shipping = get_free_shipping_or_fee(subtotal - discount)

  return subtotal + tax - discount + shipping
```

## 8. Extract Helper
Extract code with a clear purpose and limited interaction into a helper function (a.k.a. method extraction).

```
NG:
routine_a()
  unchanged_part_1()
  shared_interaction()
  unchanged_part_2()

routine_b()
  shared_interaction()
  unchanged_part_3()

OK:
helper()
  shared_interaction()

routine_a()
  unchanged_part_1()
  helper()
  unchanged_part_2()

routine_b()
  helper()
  unchanged_part_3()
```

## 9. Explicit Parameters
Split routines so required inputs are explicit parameters rather than implicit fields on a config object.

```
NG:
function process(config)
  use config.a
  use config.b  // error: undefined
  use config.x  // error: undefined

OK:
function process(config)
  process_impl(config.a, config.b, config.x)

function process_impl(a, b, x)
  // explicit parameters make requirements clear
```

## 10. Chunk Statements
Insert blank lines between logical chunks so readers see "first do A, then do B" at a glance.

```
NG:
function process(data)
  result = validate(data)
  if not result.valid
    return error
  transformed = transform(data)
  saved = save(transformed)
  notify(saved)
  return success

OK:
function process(data)
  // Validate input
  result = validate(data)
  if not result.valid
    return error

  // Transform and save
  transformed = transform(data)
  saved = save(transformed)

  // Notify completion
  notify(saved)
  return success
```

## 11. Reading Order
Place high-level logic before implementation details so readers see intent before mechanics.

```
NG:
private function helper(x)
  return x * 2

public function main()
  return helper(value)

OK:
public function main()
  return helper(value)

private function helper(x)
  return x * 2
```

## 12. Cohesion Order
Place code that changes together physically close together to reduce the cost of future changes.

```
NG:
function process_order(order)
  validate(order)
  return calculate(order)

function unrelated_function()
  // unrelated code

function validate(order)
  // validation logic

function calculate(order)
  // calculation logic

OK:
function process_order(order)
  validate(order)
  return calculate(order)

function validate(order)
  // validation logic

function calculate(order)
  // calculation logic

function unrelated_function()
  // unrelated code
```

## 13. Explaining Comments
Comment the "why" not the "what" — document context, constraints, and tradeoffs that the code cannot express.

```
NG:
// Set max to 100 if greater than 100
if max > 100 then
  max = 100

OK:
// IMPORTANT: UI component crashes with 100+ elements (bug #789).
// This is a temporary workaround until the component is fixed.
if max > 100 then
  max = 100
```
