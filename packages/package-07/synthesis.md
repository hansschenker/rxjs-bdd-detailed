# Package 07 Synthesis - Accumulation, Feedback, and Recursive Expansion

## Central finding

Package 07 shows that aggregation APIs become far simpler when decomposed into **state domain**, **transition policy**, **information retention**, and **state visibility**. Higher-order variants add an execution/ownership policy, while `expand` changes feedback from state feedback to recursively generated value feedback.

```text
ACCUMULATION / FEEDBACK POLICY
=
STATE DOMAIN
+
INITIALIZATION
+
TRANSITION FORM
+
TRANSITION EXECUTION POLICY
+
INFORMATION RETENTION
+
VISIBILITY POLICY
+
COMPLETION POLICY
```

## 1. State visibility is an independent axis

### scan

```text
input -> update state -> EMIT state -> retain state
```

### reduce

```text
input -> update state -> retain privately
...
complete -> EMIT final state
```

Both share the recurrence:

```text
S(n+1) = f(Sn, xn)
```

but differ in visibility:

```text
scan    = continuous state visibility
reduce  = terminal state visibility
```

## 2. Specialized terminal reducers

Several named operators are specialized accumulator machines:

| Operator | State | Initialization | Update | Visibility |
|---|---|---|---|---|
| `reduce` | arbitrary | seed / first value | supplied reducer | terminal |
| `count` | integer | `0` | conditional `+1` | terminal |
| `toArray` | ordered collection | `[]` | append | terminal |
| `min` | source candidate | first value | prefer lower | terminal |
| `max` | source candidate | first value | prefer higher | terminal |

The names differ, but the machine template remains stable.

## 3. Information-retention spectrum

```text
COMPLETE HISTORY
    toArray
        |
        v
QUERY-SUFFICIENT CANDIDATE
    min / max
        |
        v
SCALAR SUMMARY
    count
```

`reduce` generalizes this because its retained state is domain-defined.

This gives a useful question:

```text
What information about the source history must remain
in memory to answer the eventual query?
```

## 4. Temporal multiplicity to structural multiplicity

`toArray` changes representation:

```text
a then b then c then d
        |
        v
[a,b,c,d]
```

The values move from temporal multiplicity in the stream to spatial multiplicity in one collection.

## 5. Selection accumulation

`min` and `max` share one generic extremum machine:

```text
first value -> candidate

next value -> compare candidate and incoming

min -> prefer lower
max -> prefer higher

complete -> emit candidate
```

For structured values with a comparer, these behave conceptually like stream `argmin` / `argmax`: the selected item is retained, not merely a comparison key.

## 6. General state evolution

`scan` is naturally modeled as:

```text
Observable<Event>
      |
      v
transition(State,Event)
      |
      v
Observable<State>
```

For the seeded form:

```text
(State, Event) -> NextState
```

and the updated state is both output and future accumulator state.

This supports the architectural interpretation:

```text
event stream -> evolving state stream
```

## 7. Transition form as an independent axis

### Direct transition

```text
scan

S x E -> S
```

### Higher-order transition

```text
mergeScan / switchScan

S x E -> Observable<S>
```

This separates transition invocation from state commitment:

```text
source event
    -> start transition Observable

inner state emission
    -> commit state
```

## 8. Higher-order state ownership

`mergeScan` and `switchScan` have the same state-feedback shape but different inner ownership policies.

```text
mergeScan
    KEEP active transition work

switchScan
    CANCEL previous transition work
```

Matrix:

| Inner ownership | No state feedback | State feedback |
|---|---|---|
| KEEP | `mergeMap` | `mergeScan` |
| CANCEL previous | `switchMap` | `switchScan` |

This links Package 07 directly to the higher-order execution algebra from Package 02.

## 9. State commitment and transition ownership

For higher-order scans:

```text
TRANSITION INVOCATION
source next
    -> accumulator(state,event)
    -> inner Observable

STATE COMMITMENT
active inner next(newState)
    -> accumulated state := newState
    -> emit newState
```

`switchScan` adds:

```text
TRANSITION INVALIDATION
new source event
    -> previous transition loses ownership
```

So asynchronous state evolution can be decomposed into:

```text
INVOKE
COMMIT
INVALIDATE
```

## 10. State feedback vs value feedback

### scan family

```text
feedback carries STATE
```

```text
state + event
    -> next state
    -> feedback
```

### expand

```text
feedback carries GENERATED VALUES
```

```text
value
    -> project
    -> generated value(s)
    -> project again
```

This is the package's deepest conceptual split.

## 11. Recursive expansion topology

`expand` can generate a line:

```text
A -> B -> C -> D
```

or a tree:

```text
        A
      /   \
     B     C
    / \     \
   D   E     F
```

Every generated value is both:

```text
output
+
new expansion input
```

The computation ends when the initial source is complete and no recursive work remains.

## 12. Fold-like and unfold-like views

Package 07 provides a useful functional-programming bridge.

Fold-like consumption:

```text
scan / reduce

many source values
    -> evolving/final accumulated state
```

Unfold-like generation:

```text
expand

value
    -> generated values
    -> recursive continuation
```

This is an analogy of computational shape, not a claim that every RxJS use of these operators is a formal categorical fold or anamorphism.

The pairing becomes especially interesting when reasoning about fold/unfold pipelines and hylomorphism-inspired designs.

## 13. Memory topology summary

```text
scan / reduce
    one arbitrary state register

count
    one integer

toArray
    growing ordered history

min / max
    one candidate

mergeScan
    state register + active inner set

switchScan
    state register + replaceable active inner

expand
    active recursively generated work
```

Memory topology strongly predicts operator behavior.

## 14. Completion policies

```text
scan
    complete after source

reduce/count/toArray/min/max
    completion opens terminal visibility gate

mergeScan
    source completion plus higher-order work exhaustion

switchScan
    source completion plus currently owned inner completion

expand
    source completion plus recursive expansion exhaustion
```

Thus completion can mean:

```text
stop
publish accumulated state
or
wait for generated work to drain
```

depending on the operator policy.

## 15. Business DSL vocabulary

```text
scan        -> evolveState
reduce      -> summarize
count       -> countMatching / countAll
toArray     -> collectAll
min         -> selectMinimum
max         -> selectMaximum
mergeScan   -> evolveStateConcurrently
switchScan  -> evolveStateByLatest
expand      -> expandRecursively
```

## 16. Decision model

```text
Need each accumulated state exposed?
    -> scan

Need only a final arbitrary aggregate?
    -> reduce

Need a final quantity?
    -> count

Need the entire source history as one ordered collection?
    -> toArray

Need the smallest/largest winning source item?
    -> min / max

Need asynchronous state transitions that may coexist?
    -> mergeScan

Need newer asynchronous state transitions to replace older work?
    -> switchScan

Need generated values to recursively create further work?
    -> expand
```

## Package 07 semantic summary

```text
Accumulation is not merely "keeping a running total".

Ask:

1. What is the state domain?
2. How is initial state established?
3. How does each input update state?
4. What information is retained from history?
5. When is state visible?
6. Is a transition direct or Observable-producing?
7. If higher-order, who owns active transition work?
8. Does feedback carry accumulated state or newly generated values?
9. What condition makes the computation complete?
```

Those questions turn Package 07 from a list of aggregation operators into a reusable algebra of accumulation and feedback.
