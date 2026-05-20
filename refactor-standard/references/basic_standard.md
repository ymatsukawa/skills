# Basic standard of refactoring

## Methods of standard
1. Single Responsibility Principle
2. DRY (Don't Repeat Yourself)
3. KISS (Keep It Simple, Stupid)
4. Intention-Revealing Names
5. Self-Documenting Code
6. Parameter Count <= 3
7. Nesting Depth <= 3-4 Levels
8. 80/24 Rule

## Method-1: Single Responsibility Principle
**Each function or class should have one reason to change.**

Example:
```
NG:
class UserService:
  function saveUser(user):
    if not user.email contains "@":
      throw error "Invalid email"
    database.insert("users", user)
    emailService.send(user.email, "Welcome!")

OK:
class UserValidator:
  function validate(user):
    if not user.email contains "@":
      throw error "Invalid email"

class UserRepository:
  function save(user):
    return database.insert("users", user)
```

---

## Method-2: DRY (Don't Repeat Yourself)
**Each piece of knowledge should exist in one place.**

Example:
```
NG:
totalApples = 0
for each item in data:
  if item.kind == "apple":
    totalApples = totalApples + item.cost

totalOranges = 0
for each item in data:
  if item.kind == "orange":
    totalOranges = totalOranges + item.cost

OK:
totals = empty map
for each item in data:
  if item.kind not in totals:
    totals[item.kind] = 0
  totals[item.kind] = totals[item.kind] + item.cost
```

---

## Method-3: KISS (Keep It Simple, Stupid)
**Choose the simplest solution that works.**

Example:
```
NG:
result = arr.reduce(function(accumulator, value):
  return merge(accumulator, createMap(value.id, value)), emptyMap())

OK:
result = empty map
for each item in arr:
  result[item.id] = item
```

## Method-4: Intention-Revealing Names
**Names should communicate purpose without comments.**

Example:
```
NG:
function calc(d):
  t = d * 24
  return t

OK:
function convertDaysToHours(days):
  hoursPerDay = 24
  return days * hoursPerDay
```

## Method-5: Self-Documenting Code
**Code should explain itself; reserve comments for the *why*.**

Example:
```
NG:
// Get user by ID
function getUserById(id):
  // Query the database
  return database.find("users", id)

OK:
function findUserById(userId):
  return userRepository.findById(userId)
```

## Method-6: Parameter Count <= 3
**Keep functions at 0–3 parameters.**

Example:
```
NG:
function createUser(name, email, age, address, phone):
  user = new User()
  user.name = name
  user.email = email
  user.age = age
  user.address = address
  user.phone = phone
  return user

OK:
function createUser(userConfig):
  return new User(
    name: userConfig.name,
    email: userConfig.email,
    age: userConfig.age,
    address: userConfig.address,
    phone: userConfig.phone
  )
```

## Method-7: Nesting Depth <= 3-4 Levels
**Avoid deeply nested control structures.**

Example:
```
NG:
function processData(data):
  if data exists:                                // Level 1
    if data.isValid:                             // Level 2
      for each item in data.items:               // Level 3
        if item.active:                          // Level 4
          if item.value > 0:                     // Level 5
            process item

OK:
function processData(data):
  if not data or not data.isValid:
    return
  activeItems = data.items.filter(item => item.active and item.value > 0)
  for each item in activeItems:
    processItem(item)
```

## Method-8: 80/24 Rule
**Keep lines <= 80 chars and functions <= 24 lines.**

Example:
```
NG:
function processAndGenerateCustomerOrderReportWithValidationAndFormatting(data):
  if not data:
    throw error "No data"
  for each item in data:
    if not item.isValid:
      throw error "Invalid"
  transformedData = empty list
  for each item in data:
    transformedData.add(transform item)
  report = create new report
  report.addHeader()
  for each item in transformedData:
    report.addRow(item)
  report.addFooter()
  return report
// Violates: name > 80 chars, body grows past 24 lines once expanded

OK:
function generateReport(data):
  validated = validateData(data)
  transformed = transformData(validated)
  return writeReport(transformed)

function validateData(data):
  if not data:
    throw error "No data provided"
  for each item in data:
    if not item.isValid:
      throw error "Invalid item"
  return data
```
