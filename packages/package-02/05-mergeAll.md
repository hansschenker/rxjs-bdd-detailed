# mergeAll

## Semantic essence

`mergeAll` concurrently flattens a higher-order Observable. The source already emits inner Observables, so there is no projection step. Incoming inners are subscribed when capacity is available, existing inners remain active, and all inner emissions are merged into the sink.

## Classification

- Input: `Observable<Observable<T>>`
- Policy: concurrent merge flattening
- Memory: active inner set + optional waiting inners
- Projection: none

## Core BDD

```text
Given concurrency capacity is available
When the higher-order source emits an inner Observable
Then subscribe to that inner Observable
And keep existing inner Observables active

Given an active inner emits a value
When the value arrives
Then emit it to the sink immediately

Given concurrency capacity is full
When another inner arrives
Then defer that inner until capacity becomes available
```

## Mealy machine

```mermaid
stateDiagram-v2
    direction LR
    state "Active(I,Q)" as A
    state "SourceCompleted(I,Q)" as C
    state "Terminated" as T
    [*] --> A : subscribe higher-order source
    A --> A : inner arrives, capacity / subscribe inner
    A --> A : inner arrives, full / enqueue inner
    A --> A : inner next(x) / emit x
    A --> A : inner complete / remove inner, start queued if possible
    A --> C : source complete, work remains / defer completion
    A --> T : source complete, no work / complete sink
    C --> C : inner next(x) / emit x
    C --> C : inner complete, work remains / continue
    C --> T : all work complete / complete sink
    A --> T : source or inner error / error sink
    C --> T : inner error / error sink
    T --> [*]
```

## Marble model

```text
source       ^-- A$ ----- B$ --- C$ ---------------------|
inner A          ^-- a1 ----- a2 --------|
inner B                   ^-- b1 -- b2 --|
inner C                         ^-- c1 ------ c2 --|
sink         ^---- a1 -- b1 -- a2 -- b2 -- c1 -- c2 ----|
```

## Structural relation

```text
mergeMap(project) = map(project) + mergeAll()
```

## Domain DSL

`runAllConcurrently()`

## Semantic fingerprint

**flatten only; KEEP + START**
