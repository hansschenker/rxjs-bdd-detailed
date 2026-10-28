# combineLatestAll

## Semantic essence

`combineLatestAll` first collects inner Observables from a higher-order source. When the outer Observable completes, the participant set is finalized and latest-value coordination begins: one register is maintained per inner, all inners must initialize, and any subsequent inner update emits a complete latest snapshot.

## Classification

- Family: higher-order coordination
- Participant definition: discovered from outer source
- Participant barrier: outer completion
- Memory after discovery: latest register bank + readiness mask
- Trigger: any participant after readiness

## Core BDD

```text
Given the outer Observable has not completed
When it emits an inner Observable
Then add the inner Observable to the participant collection
And emit nothing

Given participant Observables have been collected
When the outer Observable completes
Then finalize the participant set
And subscribe to the collected inners
And initialize latest-value registers and readiness state

Given all participant registers are initialized
When participant i emits value v
Then update its register
And emit the complete latest snapshot
```

## Mealy machine

```mermaid
stateDiagram-v2
    direction LR
    state "Collecting(P)" as C
    state "Initializing(P,L,R)" as I
    state "Ready(P,L)" as R
    state "Terminated" as T
    [*] --> C : subscribe outer
    C --> C : outer next(inner) / add inner to P
    C --> I : outer complete / subscribe participants
    I --> I : inner i next(v), not all ready / update L[i], mark ready
    I --> R : inner i next(v), all become ready / emit snapshot
    R --> R : inner i next(v) / update L[i], emit snapshot
    R --> T : all participants complete / complete sink
    C --> T : outer error / error sink
    I --> T : inner error / error sink
    R --> T : inner error / error sink
    T --> [*]
```

## Structural model

```text
DISCOVER INPUTS
A$ B$ C$
    |
outer completes
    |
    v
LATEST-STATE COORDINATION
```

## Business interpretation

Discover a finite set of live state streams during setup, then continuously expose their synchronized latest state.

## Domain DSL

`latestStateOfAll()` / `combineDiscoveredLatest()`

## Semantic fingerprint

**discovered participant set + register bank + initialization barrier + any-inner trigger**
