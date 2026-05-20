# Unit test standard

## Standards
* (A) Black-box testing
  * Boundary value analysis
  * Equivalence partitioning
  * Decision table
  * State transition
  * Negative testing
* (B) White-box testing
  * Statement coverage
  * Branch coverage
  * Condition coverage
  * MC/DC

## (A) Black-box testing
### Boundary value analysis
Test values at and around each input boundary; bugs cluster at edges.

Design tests for:
* Each boundary: just-below, on, just-above
* Numeric ranges (min, max)
* String/array length (0, 1, max)
* Collection size (empty, single, full)
* Index access (first, last, out-of-range)
* Date/time edges (start, end, leap, DST)

### Equivalence partitioning
Group inputs into classes that should behave the same; test one value per class.

Design tests for:
* Each valid class (one representative value)
* Each invalid class (one representative value)
* Numeric domains split by behavior (negative / zero / positive)
* String inputs split by format (empty / valid / malformed)
* Type variants (null, missing, wrong type)

### Decision table
List combinations of conditions and their expected actions; test each rule.

Design tests for:
* Each rule (one row of the table)
* All-true combination
* All-false combination
* Mixed combinations that drive different actions
* Don't-care cells (confirm they really don't matter)

### State transition
Model the target as states and events; test each transition.

Design tests for:
* Each valid transition (state → event → next state)
* Each invalid event in a state (reject or stay)
* Initial state on construction
* Terminal state (no further transitions)
* Self-transitions and loops

### Negative testing
Feed bad input and confirm the code rejects or handles it.

Design tests for:
* Null / missing required input
* Wrong type
* Out-of-range values (overflow, underflow)
* Malformed string (empty, too long, bad format)
* Wrong call order
* External failure (I/O error, timeout)

## (B) White-box testing

### Statement coverage
Run every executable line at least once.

Design tests for:
* Every line in the target
* Both arms of if/else so each line runs
* Each case of switch/match
* Loop body (at least one iteration)
* Catch block (force the error)

### Branch coverage
Run every branch decision both ways.

Design tests for:
* if: condition true and false
* else-if chain: each arm
* switch/match: each case + default
* Loop: enter and skip
* Early return: taken and not taken

### Condition coverage
For each boolean sub-condition, make it true and false at least once.

Design tests for:
* Each atomic condition true
* Each atomic condition false
* Compound (A && B, A || B): each side independently
* Short-circuit cases (does the second operand run?)

### MC/DC
Each sub-condition independently changes the overall decision.

Design tests for:
* Each atomic condition: flip it while holding others, show the decision flips
* N+1 tests for N conditions (minimum)
* Compound expressions broken into atoms
* Skip impossible combinations and note why
