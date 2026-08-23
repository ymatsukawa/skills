---
paths:
  - "**/*"
---

# Test Design Principles
## Principles
1. Follow the test pyramid
2. Design tests against behavior

## 1. Follow the test pyramid
Taking the total number of test cases as 100%, target the following distribution:

* Unit: 60%
* Integration: 30%
* E2E: 10%

**This is a design goal, not a threshold to mechanically enforce in CI**

Divide the layers by "process" and "IO":
* Unit: Completes within its own process. Does not touch DB / files / network
* Integration: Crosses process or IO boundaries. Uses real DB / files / external processes
* E2E: Runs the whole chain through the user's In/Out

Purpose:
* Unit: To get fast feedback
* Integration: To get feedback on cases broader than Unit covers
  * However, it depends on processes and costs more to run, so keep the volume below Unit
* E2E: To get feedback from the user's point of view
  * However, it costs the most to run, so keep the volume small

## 2. Designing tests against behavior
Writing tests against "implementation details" causes the following:

* 2-1. False positives and false negatives grow
  * Because the tests trace the details
* 2-2. A bias forms that more test cases means better quality
  * Because the tests inevitably become white-box tests
* 2-3. A large body of test code is mistaken for an "asset"
  * In reality it fills up with tests that create no value for users or developers

What a test should capture is the "final result", not the "detailed process".

Therefore, write tests against "behavior".

### How to write tests against behavior

Assume the following implementation:
```text
// pseudo code
SampleLogic(a, b):
  c = a + b
  return c * 2
```

NG:
```text
// pseudo code
Test_SampleLogic_NG:
  a = 1
  b = 2
  expect = (a + b) * 2   // rebuilding the same expression as the implementation

  Expect(SampleLogic(a, b), expect)
```

OK:
```text
// pseudo code
Test_SampleLogic_OK:
  Expect(SampleLogic(1, 2), 6)
```

### Typical cases that depend on implementation details

NG-1: Verifying mock calls
```text
// pseudo code
Test_NotifyUser:
  mailer = Mock()
  NotifyUser(user, mailer)

  Expect(mailer.send.calledTimes, 1)
  Expect(mailer.send.args[0], user.email)
```

"Who called whom how many times" is a detail of collaboration. It breaks as soon as the call order changes.
What should be verified is the result — "the notification was sent" — not the path taken to get there.

NG-2: Reading private state directly
```text
// pseudo code
Test_Deposit:
  account = NewAccount()
  account.Deposit(100)

  Expect(account._balance, 100)   // reading a private field directly
```

Private fields and methods are built on the assumption that they may change at any time.
Check the result through the public API (`account.Balance()`).
