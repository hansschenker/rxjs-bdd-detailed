# Package 05 Synthesis - Lifecycle-Directed Stream Composition

## Central finding

Package 05 shows that stream composition cannot be understood from value behavior alone. Operators differ fundamentally in how they interpret subscription, completion, error, and teardown.

```text
LIFECYCLE-DIRECTED STREAM COMPOSITION
=
PARTICIPANT FORM
+
ACTIVE-SUBSCRIPTION TOPOLOGY
+
NEXT POLICY
+
COMPLETE POLICY
+
ERROR POLICY
+
TEARDOWN POLICY
```

## 1. Direct and pipeline composition matrix

| Policy | Direct | Pipeline |
|---|---|---|
| sequential success-gated | `concat` | `concatWith` |
| concurrent interleaving | `merge` | `mergeWith` |
| best-effort sequential | `onErrorResumeNext` | `onErrorResumeNextWith` |

This shows that participant form is independent of lifecycle policy.

## 2. Active-subscription topology

### Sequential

```text
A --------|
          B --------|
                    C --------|
```

Used by `concat*` and `onErrorResumeNext*`.

### Concurrent

```text
A ----------------|
   B ----------------|
      C ----------------|
```

Used by `merge*`.

The key axis is:

```text
ACTIVE-SOURCE CAPACITY

1 -------------------------------- many
|                                     |
concat                              merge
```

## 3. Completion as a policy event

The same `complete` notification means different things in different operators:

```text
concat
    complete -> advance cursor

merge
    complete -> remove participant

endWith
    complete -> activate suffix

forkJoin
    complete -> contribute to terminal barrier

onErrorResumeNext
    complete -> advance cursor

finalize
    completion teardown -> callback
```

Thus completion is not merely an endpoint. It is an input to operator policy.

## 4. Error as a policy event

```text
concat / merge / forkJoin / endWith
    error -> terminate result

onErrorResumeNext*
    error -> suppress + advance

finalize
    error -> preserve error + callback during teardown
```

This separates error propagation, error continuation, and error observation.

## 5. Lifecycle boundary decoration

### startWith

```text
SUBSCRIBE
   |
   v
PREFIX
   |
   v
SOURCE
```

### endWith

```text
SOURCE
   |
normal complete
   |
   v
SUFFIX
   |
   v
COMPLETE
```

These are specialized sequential-composition forms around source boundaries.

## 6. Lifecycle sandwich

Conceptually combining both decorators yields:

```text
PREFIX
   |
SOURCE TRACE
   |
SUFFIX
```

This separates beginning-of-lifecycle and successful-end-of-lifecycle decoration.

## 7. Terminal coordination with forkJoin

`forkJoin` uses a familiar latest-value register bank but a new trigger:

```text
participant values
    -> update latest registers

participant completion
    -> update completion mask

all emitted + all complete
    -> emit ONE final snapshot
```

Compare:

```text
combineLatest
    latest register bank + value-triggered live snapshots

forkJoin
    latest register bank + completion-triggered terminal snapshot
```

Same broad value memory, different lifecycle trigger.

## 8. Terminal-event normalization

`onErrorResumeNext*` collapse two terminal modes into one control action:

```text
complete --\
            +--> ADVANCE
error ------/
```

This is not selective recovery. Error identity does not choose the next stream; the continuation sequence is predefined.

## 9. Teardown is broader than completion

`finalize` makes subscription teardown explicit:

```text
            TERMINATION
                 |
       +---------+---------+
       |         |         |
   complete     error   unsubscribe
       |         |         |
       +---------+---------+
                 |
                 v
             finalize()
```

Therefore:

```text
notification lifecycle
!=
subscription teardown lifecycle
```

`finalize` observes the teardown path without changing source notifications.

## 10. Lifecycle-event matrix

| Operator/family | `next` | `complete` | `error` | explicit unsubscribe |
|---|---|---|---|---|
| `concat*` | forward | advance | terminate | teardown |
| `merge*` | forward | remove participant | terminate | teardown |
| `startWith` | source forward after prefix | complete normally | error normally | teardown |
| `endWith` | forward | emit suffix | terminate | teardown |
| `forkJoin` | retain latest | mark completion barrier | terminate | teardown |
| `onErrorResumeNext*` | forward | advance | suppress + advance | teardown |
| `finalize` | forward | callback on teardown | callback on teardown | callback |

## 11. Cross-package composition algebra

The active-capacity axis recurs across RxJS:

```text
Package 02
concatMap  ~= mergeMap with capacity 1
concatAll  ~= mergeAll with capacity 1

Package 05
concat     ~= merge with capacity 1
```

This suggests a reusable semantic coordinate:

```text
ACTIVE-SUBSCRIPTION CAPACITY
```

rather than unrelated sequential and concurrent APIs.

## 12. Cross-package state topology

`forkJoin` and Package 04's `combineLatest` both retain latest values:

```text
latest register bank
```

but differ in trigger ownership:

```text
combineLatest
    any value change after readiness

forkJoin
    final required completion
```

Again, the same state topology can implement different policies.

## 13. Composition vs coordination

Package 04 primarily asked:

```text
How do several participant VALUES relate?
```

Package 05 primarily asks:

```text
How do several participant LIFETIMES relate?
```

Examples:

```text
zip
    pair values by position

concat
    order entire source lifetimes

combineLatest
    synchronize current state

merge
    interleave independent lifetimes
```

## 14. Business DSL vocabulary

```text
concat                    -> runInSequence
concatWith                -> followedBy
merge                     -> interleave / unifyEvents
mergeWith                 -> interleaveWith
startWith                 -> initially / prefixWith
endWith                   -> completeWith / suffixWith
forkJoin                  -> whenAllComplete
onErrorResumeNext         -> bestEffortSequence
onErrorResumeNextWith     -> followedByBestEffort
finalize                  -> onTeardown
```

## 15. Decision model

```text
Need streams strictly one after another?
    -> concat / concatWith

Need streams active together with direct interleaving?
    -> merge / mergeWith

Need configured initial output before the source?
    -> startWith

Need configured output after normal completion?
    -> endWith

Need one final result after every finite participant completes?
    -> forkJoin

Need later stages attempted even if a stage errors?
    -> onErrorResumeNext / onErrorResumeNextWith

Need cleanup whenever the subscription ends?
    -> finalize
```

## Package 05 semantic summary

```text
A stream operator's behavior is not defined by next-values alone.

Ask:

1. Who is subscribed, and when?
2. How many sources may be active?
3. What happens on next?
4. What does completion mean?
5. What does error mean?
6. What happens on explicit teardown?
```

That is the core conceptual map of Package 05.
