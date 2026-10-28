# switchScan

## Semantic essence

`switchScan` combines accumulated-state feedback with switch-style higher-order execution. A new source value replaces the previously active transition Observable; emissions from the current transition become both output and accumulated state.

## Core BDD

```text
Given accumulated state s exists
When source value v arrives
Then cancel the previous transition Observable if active
And project (s,v,index) to a new transition Observable
And subscribe to that Observable

Given the active transition emits s'
Then emit s'
And set accumulated state := s'
```

## Mealy machine

```mermaid
stateDiagram-v2
    state "Running(s,inner?)" as R
    state "Terminated" as T
    [*] --> R : initialize seed
    R --> R : source next(v) / cancel old inner, project and subscribe new inner
    R --> R : active inner next(s') / s := s', emit s'
    R --> R : active inner complete / clear inner
    R --> T : source complete and no active inner / complete
    R --> T : source or active-inner error / error
```

## Domain DSL

`evolveStateByLatest(...)`

## Semantic fingerprint

**state feedback + Observable transition + CANCEL previous work**
