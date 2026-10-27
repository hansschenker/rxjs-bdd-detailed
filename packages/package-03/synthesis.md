# Package 03 Synthesis - Segmentation and Representation Algebra

## Central finding

Package 03 reduces ten RxJS operators to **five segmentation policies crossed with two representation policies**.

```text
SEGMENT OPERATOR
    =
SEGMENTATION POLICY
    x
REPRESENTATION POLICY
```

## 1. The 5 x 2 matrix

| Segmentation policy | Materialized | Streaming |
|---|---|---|
| notifier boundary | `buffer` | `window` |
| count geometry | `bufferCount` | `windowCount` |
| temporal geometry | `bufferTime` | `windowTime` |
| independent open/close lifecycle | `bufferToggle` | `windowToggle` |
| dynamic sequential partition | `bufferWhen` | `windowWhen` |

Representation:

```text
BUFFER FAMILY
    segment -> Array<T>

WINDOW FAMILY
    segment -> Observable<T>
```

## 2. Four segmentation coordinates

```text
SEGMENTATION POLICY
    =
OPEN
+
MEMBERSHIP
+
CLOSE
+
REPRESENT
```

### OPEN

When does a new segment begin?

```text
immediately
external notifier lifecycle
count schedule
time schedule
arbitrary opening Observable
immediately after previous close
```

### MEMBERSHIP

Which active segments receive a source value?

```text
one current segment
or
every currently active segment
```

### CLOSE

What ends a segment?

```text
external notifier
count reached
time elapsed
max size/count
per-opening closing Observable
dynamically regenerated closing Observable
```

### REPRESENT

How is the segment exposed?

```text
Array<T>
Observable<T>
```

## 3. Materialized vs streaming segments

### Buffer

```text
source values
    |
    v
collect internally
    |
segment closes
    |
    v
emit Array<T>
```

The segment is available collectively at closure.

### Window

```text
segment opens
    |
    v
emit Observable<T>
    |
source values flow through it while active
    |
segment closes
    |
complete inner Observable
```

The segment is available incrementally while alive.

## 4. Segment geometry

Count- and time-based operators reveal the same geometry:

```text
opening spacing < segment extent
    -> overlap

opening spacing = segment extent
    -> adjacent segments

opening spacing > segment extent
    -> gaps
```

This geometry is independent of representation.

## 5. Partition vs cover

### Partition-like policies

Exactly one current segment during normal operation:

```text
buffer
bufferWhen
window
windowWhen
```

Conceptually:

```text
|----S1----|----S2----|----S3----|
```

### Cover-like policies

Zero, one, or multiple active segments are possible:

```text
bufferToggle
windowToggle
```

and overlapping count/time configurations.

Conceptually:

```text
S1  [----------------]
S2       [----------------]
S3             [-----------]
```

## 6. Membership multiplicity

A source value may belong to:

```text
0 segments
1 segment
N segments
```

depending on segmentation geometry.

This gives Package 03 a useful axis:

```text
MEMBERSHIP MULTIPLICITY
0 / 1 / many
```

## 7. Dynamic-set machines

`bufferToggle` and `windowToggle` maintain sets of independently created segment contexts.

```text
opening(o)
    -> add segment

source next(v)
    -> route v to every active segment

closing(i)
    -> remove segment i
```

Buffer contexts retain arrays.
Window contexts expose live streams.

## 8. Dynamic sequential machines

`bufferWhen` and `windowWhen` use one current segment and regenerate the closing mechanism after every close.

```text
Segment n + Boundary n
        |
        | boundary fires
        v
Segment n+1 + Boundary n+1
```

This creates a continuous dynamic partition rather than a dynamic cover.

## 9. Competing boundaries

Time-based operators may combine multiple closing conditions:

```text
TIME EXPIRES
      OR
MAX SIZE / COUNT REACHED
      |
      v
segment closes
```

This is a reusable concept beyond buffering/windowing: a state may be exited by the first of several competing boundary events.

## 10. Data-flow topology

Package 03 complements Package 02.

### Package 02 merge-style fan-in

```text
inner 1 --\
inner 2 ---+--> sink
inner 3 --/
```

### Package 03 overlapping-segment fan-out

```text
           /--> segment 1
source ---+---> segment 2
           \--> segment 3
```

## 11. Cross-package relationship

Window operators create higher-order Observables:

```text
window*
    |
    v
Observable<Observable<T>>
```

Package 02 then supplies flattening policies:

```text
mergeAll
concatAll
switchAll
exhaustAll
```

This creates a natural curriculum sequence:

```text
higher-order flattening
        ->
window generation
        ->
per-window transformation
        ->
flattening choice
```

## 12. Business-language DSL vocabulary

```text
buffer        -> batchUntil / collectUntil
bufferCount   -> batchEvery / slidingWindow
bufferTime    -> batchByTime
bufferToggle  -> collectBetween
bufferWhen    -> collectByDynamicBoundary

window        -> segmentBy
windowCount   -> segmentEvery / streamGroupsOf
windowTime    -> timeWindows
windowToggle  -> streamBetween
windowWhen    -> streamByDynamicBoundary
```

## 13. Package 03 decision model

```text
Do you need complete groups as arrays?
|
+-- YES -> BUFFER representation
|
+-- NO, process while segment is alive -> WINDOW representation

Then choose segmentation policy:

external boundary        -> buffer / window
count                     -> bufferCount / windowCount
time                      -> bufferTime / windowTime
independent open + close  -> bufferToggle / windowToggle
dynamic sequential close -> bufferWhen / windowWhen
```

## 14. Package 03 semantic summary

```text
Buffering and windowing are two representations of the same deeper idea:
source-trace segmentation.

The important questions are:

1. When does a segment open?
2. Which active segments receive each source value?
3. What closes each segment?
4. May segments overlap or leave gaps?
5. Is the segment materialized as an array or exposed as a live stream?
```

That is the core conceptual map of Package 03.
