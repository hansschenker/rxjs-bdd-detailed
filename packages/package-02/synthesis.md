# Package 02 Synthesis - Higher-Order Policy Algebra

## Central finding

Package 02 shows that the core higher-order RxJS operators are not ten unrelated APIs. Eight collapse into **four flattening policies crossed with two inner-origin forms**, while two additional operators introduce distinct feedback topologies.

## 1. Four flattening policies

```text
MERGE    = KEEP current + START new
CONCAT   = KEEP current + QUEUE new
SWITCH   = CANCEL current + START new
EXHAUST  = KEEP current + DROP new
```

## 2. Two structural forms

| Policy | Project + flatten | Flatten only |
|---|---|---|
| MERGE | `mergeMap` | `mergeAll` |
| CONCAT | `concatMap` | `concatAll` |
| SWITCH | `switchMap` | `switchAll` |
| EXHAUST | `exhaustMap` | `exhaustAll` |

The structural decomposition is:

```text
source<T>
    |
    v
map(project)              optional
    |
    v
Observable<Observable<R>>
    |
    v
flatten(policy)
    |
    v
Observable<R>
```

Thus:

```text
mergeMap(project)   = map(project) + mergeAll()
concatMap(project)  = map(project) + concatAll()
switchMap(project)  = map(project) + switchAll()
exhaustMap(project) = map(project) + exhaustAll()
```

## 3. Inner-lifetime topology

### MERGE

```text
A ----------------|
    B ----------------|
        C ----------------|
```

Multiple inners overlap.

### CONCAT

```text
A --------|
          B --------|
                    C --------|
```

Inners serialize through a FIFO queue.

### SWITCH

```text
A ------x
       B -----x
            C ------------|
```

The newest inner replaces the previous inner.

### EXHAUST

```text
A ----------------|
    B x
        C x
                  D --------|
```

The active inner is protected; newcomers are dropped while busy.

## 4. Higher-order work-admission matrix

| Existing work | New work behavior | Policy |
|---|---|---|
| keep | start | MERGE |
| keep | queue | CONCAT |
| cancel | start replacement | SWITCH |
| keep | drop | EXHAUST |

This reframes flattening as **admission control over computations**.

Package 01 controlled admission of values. Package 02 controls admission and lifetime of inner computations.

## 5. Subscription memory topologies

```text
merge      active subscription set
concat     one active slot + FIFO queue
switch     one replaceable subscription slot
exhaust    one protected subscription slot
```

## 6. Concurrency-capacity axis

`mergeMap` and `mergeAll` expose configurable concurrency. The serialized concat policy corresponds conceptually to merge concurrency constrained to one active inner.

```text
1 ----------------------------------------- Infinity
|                                               |
serialized                                   concurrent
concat                                       merge
```

## 7. Lifecycle principle

Across the higher-order family:

```text
outer/source completion
        !=
immediate sink completion
```

when admitted inner work remains.

The operator normally completes only after the outer can no longer produce work and the relevant admitted inner work has finished.

By contrast, error or downstream unsubscription terminates/cancels active computation according to lifecycle semantics.

## 8. Feedback operators

### mergeScan - state feedback

```text
source value
    +
current accumulator
    |
    v
Observable<next state>
    |
    +----> sink
    |
    +----> accumulator state
```

Conceptual transition:

```text
S x A -> Observable<S>
```

### expand - work-generation feedback

```text
value
  |
  +----> sink
  |
  +----> project(value)
             |
             v
        successor value
             |
             +----> sink
             +----> project again
```

`mergeScan` feeds inner results back into **state**.

`expand` feeds generated values back into **projection/work generation**.

## 9. Decision tree for the four mapping policies

```text
Does every source-triggered operation matter?
|
+-- YES
|   |
|   +-- May operations overlap?
|       |
|       +-- YES -> mergeMap
|       |
|       +-- NO  -> concatMap
|
+-- NO
    |
    +-- Should newer work replace existing work?
        |
        +-- YES -> switchMap
        |
        +-- NO, protect current work -> exhaustMap
```

## 10. Business-language DSL vocabulary

```text
mergeMap      -> runConcurrently(project)
concatMap     -> runSequentially(project)
switchMap     -> latestWins(project)
exhaustMap    -> ignoreWhileBusy(project)

mergeAll      -> runAllConcurrently()
concatAll     -> runSequentially()
switchAll     -> observeLatest()
exhaustAll    -> protectCurrentWork()

mergeScan     -> evolveAsync(seed, transition)
expand        -> unfoldAsync(project)
```

## 11. Cross-package connections

### Package 01 audit vs Package 02 switch

Both can use a single-slot replacement topology, but at different levels:

```text
audit      replaces pending DATA
switchAll  replaces current COMPUTATION
```

### Package 01 throttle vs Package 02 exhaust

Both express a first-wins-while-busy policy:

```text
throttle    busy = duration active
exhaust     busy = inner computation active
```

## 12. Teaching progression

```text
Map to inner Observables
        |
        v
Expose higher-order Observable
        |
        v
Choose flattening policy
        |
        +-- MERGE
        +-- CONCAT
        +-- SWITCH
        +-- EXHAUST
        |
        v
Reason about inner lifetime topology
        |
        v
Reason about completion, cancellation, queueing, and concurrency
        |
        v
Extend to state feedback and recursive work generation
```

## Package 02 semantic summary

```text
Higher-order RxJS is largely a policy language for deciding:

1. where inner computations come from,
2. whether current computations are preserved or cancelled,
3. whether new computations start, wait, or disappear,
4. how many computations may be active,
5. when the sink may complete,
6. whether inner results feed back into state or future work.
```

That is the core conceptual map of Package 02.
