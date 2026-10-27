# buffer

## Semantic essence

`buffer` maintains one current collection. Every source value is appended to that collection. Whenever the closing notifier emits, the current collection is emitted as an array, cleared, and collection continues in a new empty buffer.

## Classification

- Family: buffering / batching
- State: one current `Array<T>`
- Opening: continuous
- Closing: external notifier
- Membership: each source value belongs to the current buffer
- Representation: `T[]`

## Core BDD

```text
Given a buffer is open
When a source value arrives
Then append the source value to the current buffer
And emit nothing to the sink

Given a current buffer exists
When the closing notifier emits
Then emit the current buffer to the sink
And replace it with a new empty buffer
```

## Mealy machine

```mermaid
stateDiagram-v2
    direction LR
    state "Collecting(B)" as C
    state "Terminated" as T
    [*] --> C : subscribe / B := empty
    C --> C : source next(v) / append v to B
    C --> C : notifier next / emit B, B := empty
    C --> T : source complete / resolve B, complete sink
    C --> T : source error / discard B, error sink
    C --> T : unsubscribe / discard B
    T --> [*]
```

## Marble model

```text
source       ^-- a -- b -- c -------- d -- e ------ f --|
buffer           [a] [a,b] [a,b,c]    [d] [d,e]    [f]
notifier     ^--------------!---------------!-------------
sink         ^----------[a,b,c]-----------[d,e]-----------|
```

## Business interpretation

Batch analytics or domain events until an external flush signal occurs.

## Domain DSL

`batchUntil(flush$)` / `collectUntil(boundary$)`

## Semantic fingerprint

**collect all + external boundary + emit array + reset**
