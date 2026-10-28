# zipAll

## Semantic essence

`zipAll` collects participant Observables from a higher-order source. When the outer source completes, it subscribes to the finalized participant set and performs positional rendezvous using one FIFO queue per participant.

## Classification

- Participant definition: discovered from higher-order source
- Participant barrier: outer completion
- Coordination: positional rendezvous
- Memory after discovery: FIFO queue per participant

## Core BDD

```text
Given the outer Observable has not completed
When it emits an inner Observable
Then add that inner to the participant collection

Given participants have been collected
When the outer completes
Then subscribe to every collected participant
And create one FIFO queue per participant

Given every queue is nonempty
When the next positional tuple is available
Then dequeue one value from every queue
And emit the tuple
```

## Mealy machine

```mermaid
stateDiagram-v2
    direction LR
    state "Collecting(P)" as C
    state "Coordinating(Q)" as Z
    state "Terminated" as T
    [*] --> C : subscribe outer
    C --> C : outer next(inner) / add inner to P
    C --> Z : outer complete / subscribe participants, create queues
    Z --> Z : inner next(v), not all queues ready / enqueue
    Z --> Z : inner next(v), all queues ready / enqueue, dequeue one each, emit
    Z --> T : next full tuple impossible / complete sink
    C --> T : outer error / error sink
    Z --> T : inner error / error sink
    T --> [*]
```

## Business interpretation

Discover a finite set of ordered worker streams, then combine their corresponding results by position.

## Domain DSL

`rendezvousAll()`

## Semantic fingerprint

**discovered participant set + FIFO queue bank + positional consumption**
