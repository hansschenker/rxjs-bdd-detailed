# count

## Semantic essence

`count` is specialized terminal accumulation with an integer state. Every source value, or every value satisfying an optional predicate, increments the private counter. Completion exposes the final count.

## Core BDD

```text
Given current count is n
When source value v arrives
Then evaluate the optional predicate
And if v qualifies set count := n + 1
And otherwise retain n
And emit nothing

When the source completes
Then emit the final count
And complete
```

## Mealy machine

```mermaid
stateDiagram-v2
    state "Count(n)" as C
    state "Terminated" as T
    [*] --> C : n := 0
    C --> C : next(v), qualifies / n := n + 1
    C --> C : next(v), not qualifying / n unchanged
    C --> T : complete / emit n, complete
    C --> T : error or predicate throws / error
```

## Formal view

```text
count(P) = sum_i indicator(P(x_i))
```

## Domain DSL

`countMatching(predicate)` / `countAll()`

## Semantic fingerprint

**qualification + private integer accumulation + terminal visibility**
