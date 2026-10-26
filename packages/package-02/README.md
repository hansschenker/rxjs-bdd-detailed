# Package 02 - Higher-Order Mapping, Flattening, and Feedback

Package 02 of **RxJS BDD Detailed** develops the core higher-order concurrency policies and then extends the model with state feedback and recursive work generation.

## Operators

1. `mergeMap`
2. `concatMap`
3. `switchMap`
4. `exhaustMap`
5. `mergeAll`
6. `concatAll`
7. `switchAll`
8. `exhaustAll`
9. `mergeScan`
10. `expand`

## Central teaching model

Eight of the operators collapse into four flattening policies crossed with two inner-origin forms:

| Flattening policy | Project + flatten | Flatten only |
|---|---|---|
| MERGE | `mergeMap` | `mergeAll` |
| CONCAT | `concatMap` | `concatAll` |
| SWITCH | `switchMap` | `switchAll` |
| EXHAUST | `exhaustMap` | `exhaustAll` |

The policy vocabulary is:

```text
MERGE    = KEEP current + START new
CONCAT   = KEEP current + QUEUE new
SWITCH   = CANCEL current + START new
EXHAUST  = KEEP current + DROP new
```

The final two operators add feedback:

```text
mergeScan = inner result -> accumulator state
expand    = inner result -> new projection input
```

See `synthesis.md` for the package-level behavioral model.
