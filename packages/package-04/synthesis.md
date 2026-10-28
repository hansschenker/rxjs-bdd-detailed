# Package 04 Synthesis - Coordination Policy Algebra

## Central finding

Package 04 demonstrates that multi-stream combination operators are best understood through **coordination policies**, not through a flat list of API names.

```text
COORDINATION POLICY
=
PARTICIPANT DEFINITION
+
MEMORY TOPOLOGY
+
READINESS RULE
+
TRIGGER OWNERSHIP
+
OUTPUT ASSEMBLY
+
SELECTION / CONSUMPTION POLICY
```

## 1. Five coordination families

### Latest-state synchronization

```text
combineLatest
combineLatestAll
combineLatestWith
```

Memory:

```text
latest-value register bank
```

Policy:

```text
wait until all initialized
then any participant change
-> emit complete current snapshot
```

### Triggered latest snapshot

```text
withLatestFrom
```

Memory:

```text
latest companion register bank
```

Policy:

```text
companions update memory only
source owns output trigger
```

### Positional rendezvous

```text
zip
zipAll
zipWith
```

Memory:

```text
FIFO queue per participant
```

Policy:

```text
wait until every queue nonempty
consume one value from each
emit positional tuple
```

### Competitive selection

```text
race
raceWith
```

Memory topology:

```text
candidate set
    ->
winner identity
```

Policy:

```text
one-shot selection
losers cease participation
follow winner
```

### Local temporal coordination

```text
pairwise
```

Memory:

```text
one previous-value register
```

Policy:

```text
previous + current
-> transition tuple
```

## 2. Structural participant forms

Two major coordination policies appear in three participant forms:

| Coordination | Direct | Discovered | Pipeline |
|---|---|---|---|
| Latest | `combineLatest` | `combineLatestAll` | `combineLatestWith` |
| Positional | `zip` | `zipAll` | `zipWith` |

Competitive selection appears in direct and pipeline forms:

| Coordination | Direct | Pipeline |
|---|---|---|
| Competition | `race` | `raceWith` |

This shows that **participant definition is independent of coordination policy**.

## 3. Memory topology generates semantics

### Register bank

```text
A [latest]
B [latest]
C [latest]
```

Used by latest-state synchronization.

Old values are replaced. Current state matters.

### Companion register bank

```text
B [latest]
C [latest]

A = transient trigger
```

Used by `withLatestFrom`.

### Queue bank

```text
A [a1][a2][a3]
B [b1][b2]
C [c1]
```

Used by positional rendezvous.

Unmatched values are retained until consumed.

### Candidate set to winner

```text
{A,B,C}
   |
   v
   B
```

Used by competition.

### Previous register

```text
[previous]
```

Used by `pairwise`.

## 4. Readiness policies

```text
combineLatest*
    every latest register initialized

withLatestFrom
    every companion register initialized
    AND source event occurs

zip*
    every FIFO queue currently nonempty

race*
    competition resolved by decisive participant notification

pairwise
    previous value exists
```

Readiness is therefore not a universal Boolean concept. Its meaning is policy-specific.

## 5. Trigger ownership

### Symmetric trigger ownership

```text
combineLatest*
```

After readiness, any participant can trigger output.

### Asymmetric trigger ownership

```text
withLatestFrom
```

Only the pipeline source triggers output. Companion streams update context only.

### Rendezvous trigger

```text
zip*
```

The event that makes every queue nonempty completes the next tuple.

### Competitive trigger

```text
race*
```

The first decisive participant notification resolves selection.

### Local trigger

```text
pairwise
```

Every source value after the first triggers a transition tuple.

## 6. State vs event

`withLatestFrom` gives an especially useful FRP distinction:

```text
SOURCE
    event stream

COMPANIONS
    state/context streams
```

Then:

```text
event + current state -> enriched event
```

This is a general architecture pattern for commands and user actions.

## 7. Snapshot vs correspondence

### Latest operators

```text
What is everybody's current state?
```

Values can be reused across multiple outputs.

### Zip operators

```text
What are everybody's next corresponding values?
```

Values are consumed once.

This gives a useful axis:

```text
REUSABLE STATE <----------------> CONSUMPTIVE CORRESPONDENCE
```

## 8. Coordination feasibility and completion

Several operators expose a deeper lifecycle principle:

```text
Can the coordination invariant still be satisfied in the future?
```

Examples:

```text
combineLatest:
    if an input completes without ever supplying a value,
    full readiness is impossible

zip:
    if a participant is complete and has no value available
    for the next index, another tuple is impossible
```

Thus completion may follow from **future coordination impossibility** rather than merely the occurrence of one completion event.

## 9. Higher-order participant discovery

`combineLatestAll` and `zipAll` share a structural first phase:

```text
outer emits participant Observables
        |
        v
collect participant set
        |
outer completes
        |
        v
finalize participant topology
```

Then they diverge:

```text
combineLatestAll -> latest register bank
zipAll           -> FIFO queue bank
```

This demonstrates that participant discovery and coordination semantics are orthogonal.

## 10. Competition topology

```text
A --\
B ---+--> competition --> winner --> sink
C --/
```

After selection:

```text
losers stop participating
winner becomes output authority
```

This differs from both synchronization and rendezvous because participant multiplicity collapses from many candidates to one authoritative stream.

## 11. Cross-package connection: pairwise and Package 03

`pairwise` has the same trace geometry as a sliding count segment of size two and step one:

```text
[a,b]
   [b,c]
      [c,d]
```

Package 03 interprets this as overlapping segmentation.

Package 04 interprets it as local previous/current coordination.

Same geometry, different semantic policy.

## 12. Cross-package connection: pairwise and distinctUntilChanged

Both use a previous-value register.

```text
distinctUntilChanged
    previous + current -> compare -> admit/suppress

pairwise
    previous + current -> expose transition pair
```

Again, the same memory topology can support different action policies.

## 13. Business-language DSL vocabulary

```text
combineLatest      -> latestStateOf / synchronizeLatest
combineLatestAll   -> latestStateOfAll
combineLatestWith  -> synchronizeWithLatest
withLatestFrom     -> snapshotWithLatest / enrichWithLatest

zip                -> rendezvous / alignByPosition
zipAll             -> rendezvousAll
zipWith            -> rendezvousWith / alignWith

race               -> firstToEmit / compete
raceWith           -> raceAgainst / firstSignalWins

pairwise           -> transitions
```

## 14. Decision model

```text
Need current state from all participants?
    -> combineLatest family

Need one event enriched with current companion state?
    -> withLatestFrom

Need one-to-one ordered correspondence?
    -> zip family

Need one competing source to become authoritative?
    -> race family

Need previous/current relation inside one stream?
    -> pairwise
```

## 15. Package 04 semantic summary

```text
Combination is not one behavior.

Coordination semantics depend on:

1. who participates,
2. what state is retained for each participant,
3. what makes the system ready,
4. which event owns the output trigger,
5. whether values are reused or consumed,
6. whether all participants remain relevant or one winner is selected.
```

That is the core conceptual map of Package 04.
