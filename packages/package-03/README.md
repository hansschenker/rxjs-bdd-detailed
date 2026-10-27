# Package 03 - Segmentation, Buffering, and Windowing

Package 03 of **RxJS BDD Detailed** analyzes ten RxJS segmentation operators as five segmentation policies crossed with two representation policies.

## Operators

1. `buffer`
2. `bufferCount`
3. `bufferTime`
4. `bufferToggle`
5. `bufferWhen`
6. `window`
7. `windowCount`
8. `windowTime`
9. `windowToggle`
10. `windowWhen`

## Central teaching model

```text
SEGMENT OPERATOR
    =
SEGMENTATION POLICY
    x
REPRESENTATION POLICY
```

| Segmentation policy | Materialized | Streaming |
|---|---|---|
| notifier | `buffer` | `window` |
| count | `bufferCount` | `windowCount` |
| time | `bufferTime` | `windowTime` |
| toggle | `bufferToggle` | `windowToggle` |
| dynamic sequential | `bufferWhen` | `windowWhen` |

Representation:

```text
BUFFER -> Array<T>
WINDOW -> Observable<T>
```

See `synthesis.md` for the package-level model.
