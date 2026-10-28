# min

## Semantic essence

`min` maintains one smallest-so-far candidate according to its comparison policy. Intermediate candidates remain private; completion emits the surviving minimum.

## Core BDD

```text
Given no candidate exists
When the first source value arrives
Then retain it as the candidate

Given candidate m exists
When source value v arrives
Then retain the smaller of m and v according to the comparer
And emit nothing

When the source completes
Then emit the retained candidate
And complete
```

## Mealy machine

```mermaid
stateDiagram-v2
    state "NoCandidate" as N
    state "Candidate(m)" as C
    state "Terminated" as T
    [*] --> N
    N --> C : first next(v) / m := v
    C --> C : next(v) / retain preferred minimum
    C --> T : complete / emit m, complete
    C --> T : error or comparer throws / error
```

## Domain DSL

`selectMinimum(comparer)`

## Semantic fingerprint

**one query-sufficient candidate + prefer lower + terminal visibility**
