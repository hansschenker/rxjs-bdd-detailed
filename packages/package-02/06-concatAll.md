# concatAll

## Semantic essence

`concatAll` serially flattens a higher-order Observable. It subscribes to one inner at a time, queues later inner Observables in arrival order, and advances to the next only when the current inner completes.

## Classification

- Input: `Observable<Observable<T>>`
- Policy: serialized flattening
- Memory: one active inner + FIFO inner queue
- Projection: none

## Core BDD

```text
Given no inner Observable is active
When an inner Observable arrives
Then subscribe immediately

Given an inner Observable is active
When another inner arrives
Then keep the active inner running
And enqueue the new inner

Given the active inner completes
And the queue is not empty
When completion occurs
Then dequeue and subscribe to the oldest waiting inner
```

## Mealy machine

```mermaid
stateDiagram-v2
    direction LR
    state "Idle" as I
    state "Active(inner, queue)" as A
    state "SourceCompleted(inner, queue)" as C
    state "Terminated" as T
    [*] --> I : subscribe higher-order source
    I --> A : inner arrives / subscribe inner
    A --> A : inner arrives / enqueue inner
    A --> A : active inner next(x) / emit x
    A --> A : active inner complete, queue not empty / subscribe next queued inner
    A --> I : active inner complete, queue empty / return idle
    I --> T : source complete / complete sink
    A --> C : source complete / defer completion
    C --> C : active inner next(x) / emit x
    C --> C : inner complete, queue not empty / subscribe next queued inner
    C --> T : inner complete, queue empty / complete sink
    A --> T : source or inner error / error sink
    C --> T : inner error / error sink
    T --> [*]
```

## Marble model

```text
source       ^-- A$ -- B$ -- C$ -------------------------|
queue            []  [B] [B,C]
inner A          ^-- a1 -- a2 -- a3 --|
inner B                               ^-- b1 -- b2 --|
inner C                                             ^-- c1 --|
sink         ^---- a1 -- a2 -- a3 ------ b1 -- b2 ------ c1 --|
```

## Structural relation

```text
concatMap(project) = map(project) + concatAll()
concatAll() conceptually occupies mergeAll(concurrency = 1)
```

## Domain DSL

`runSequentially()`

## Semantic fingerprint

**flatten only; KEEP + QUEUE**
