# Package 04 - Combination and Coordination

Package 04 of **RxJS BDD Detailed** analyzes multi-stream and local coordination through participant definition, memory topology, readiness, trigger ownership, output assembly, and selection or consumption policy.

## Operators

1. `combineLatest`
2. `combineLatestAll`
3. `combineLatestWith`
4. `withLatestFrom`
5. `zip`
6. `zipAll`
7. `zipWith`
8. `race`
9. `raceWith`
10. `pairwise`

## Central teaching model

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

Core families:

```text
LATEST SYNCHRONIZATION -> register bank -> complete current snapshot
TRIGGERED LATEST      -> companion registers -> source-triggered snapshot
POSITIONAL RENDEZVOUS -> FIFO queue bank -> consumptive tuple
COMPETITION           -> candidate set -> one-shot winner
LOCAL COORDINATION    -> previous register -> [previous,current]
```

See `synthesis.md` for the package-level model.
