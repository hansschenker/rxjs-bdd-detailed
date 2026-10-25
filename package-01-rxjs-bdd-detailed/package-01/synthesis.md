# Package 01 Synthesis

## Central finding

The ten operators are easier to understand as **admission and observation policies** than as ten unrelated API names.

## Four temporal policy pairs

| Behavioral policy | Dynamic / Observable control | Fixed-time control |
|---|---|---|
| Retain latest, keep boundary | `audit` | `auditTime` |
| Retain latest, replace/restart boundary | `debounce` | `debounceTime` |
| Admit leading, suppress remainder | `throttle` | `throttleTime` |
| Retain latest, external observation | `sample` | `sampleTime` |

## Value-history pair

| Operator | Memory topology | Admission meaning |
|---|---|---|
| `distinct` | set of all seen keys | global novelty |
| `distinctUntilChanged` | one previous admitted key | local change |

## Semantic axes discovered

### 1. Boundary source

```text
Observable-defined <--------------------> fixed time
```

### 2. Boundary reaction to source values

```text
persistent <----------------------------> restarting
```

### 3. Boundary ownership

```text
source-triggered <----------------------> independent observer
```

### 4. Clock topology

```text
event-relative <------------------------> periodic subscription-relative
```

### 5. Representative selection

```text
leading <-------------------------------> trailing
```

### 6. Memory topology

```text
none | one register | pending slot | history set
```

### 7. Admission reason

```text
audit       : boundary not reached yet
debounce    : value not stable yet
throttle    : gate closed
sample      : observation not requested yet
distinct    : key already seen
distinctUntilChanged : key has not changed locally
```

## Reusable machine patterns

### Pending-slot machine

Used by `audit`, `auditTime`, `debounce`, `debounceTime`.

```text
source -> pending -> release policy -> sink
```

### Gate machine

Used by default `throttle` / `throttleTime`.

```text
OPEN -> admit -> CLOSED -> suppress -> boundary -> OPEN
```

### Snapshot machine

Used by `sample` / `sampleTime`.

```text
source updates latest state
independent observer decides when to expose it
```

### History-set machine

Used by `distinct`.

```text
key -> membership test -> admit-and-record OR suppress
```

### Comparison-register machine

Used by `distinctUntilChanged`.

```text
current key -> compare previous admitted key -> suppress OR emit+replace
```

## Business-language DSL vocabulary

```text
audit / auditTime             -> requestEvery / rate-controlled trailing release
debounce / debounceTime       -> afterStable / afterStableFor
throttle / throttleTime       -> admitThenSuppress / admitThenSuppressFor
sample / sampleTime           -> snapshotWhen / snapshotEvery
distinct                      -> admitOnce / admitOnceBy
distinctUntilChanged          -> whenChanged / whenKeyChanged
```

## Teaching model

```text
BDD scenarios
      |
      v
Mealy machine
      |
      v
Marble trace
      |
      v
Business interpretation
      |
      v
Domain-specific DSL
```

The marble trace demonstrates an execution. The Mealy machine explains the state transitions that produce that execution. The BDD rules provide the readable behavioral specification.
