# Package 06 Synthesis - Temporal Semantics and Scheduling

## Central finding

Package 06 shows that RxJS time semantics are not one concept. Time can be used to **move delivery**, **control execution placement**, **enforce temporal requirements**, or **be observed as data**.

```text
TEMPORAL SEMANTICS
=
TEMPORAL SUBJECT
+
REFERENCE
+
CLOCK / SCHEDULER
+
STATE TOPOLOGY
+
TRIGGER
+
ACTION
```

## 1. The four temporal roles

```text
                         TIME

                          |
       +------------------+------------------+
       |                  |                  |
       v                  v                  v

     PLACE             SUPERVISE           OBSERVE
       |                  |                  |
   delay              timeout           timestamp
   delayWhen                            timeInterval

                          |
                          v

                       SCHEDULE
                          |
                     observeOn
                     subscribeOn
```

A clearer behavioral grouping is:

```text
TEMPORAL PLACEMENT
    delay
    delayWhen

SCHEDULER PLACEMENT
    observeOn
    subscribeOn

TEMPORAL SUPERVISION
    timeout

TEMPORAL MEASUREMENT
    timestamp
    timeInterval
```

## 2. Occurrence time vs delivery time

`delay` establishes the distinction:

```text
source occurrence time
        !=
downstream delivery time
```

For a fixed translation:

```text
t_i -> t_i + Delta
```

The value itself remains unchanged.

## 3. Translation vs gating

### delay

```text
v -> fixed/absolute temporal placement -> v
```

The same broad timing policy applies to all values.

### delayWhen

```text
v -> durationSelector(v) -> Dv$ -> release v on Dv next
```

Each value receives its own temporal release lifecycle.

This gives:

```text
delay      = temporal translation
delayWhen  = temporal gating
```

## 4. Pending-state topology

### delay

```text
{ scheduled(v1), scheduled(v2), ... }
```

### delayWhen

```text
{ (v1,D1), (v2,D2), ... }
```

### timeout

```text
pending EXPECTATION / armed deadline
```

This contrasts pending data with pending temporal requirements.

## 5. Subscription vs notification direction

RxJS runtime behavior has two conceptual directions:

```text
SUBSCRIPTION
    downstream -> upstream

NOTIFICATIONS
    upstream -> downstream
```

That gives the scheduler pair:

```text
subscribeOn
    schedules the subscription action

observeOn
    schedules downstream notifications
```

Diagram:

```text
                 subscription
subscriber --------------------------> source
                    ^
               subscribeOn

source ------------------------------> subscriber
                notifications
                    |
                observeOn
```

## 6. Scheduler boundary is not source-scheduler replacement

`observeOn` should be understood as a notification handoff:

```text
source execution
      |
      v
notification
      |
      v
observeOn(S)
      |
      v
downstream delivery on S
```

The source retains its own execution/scheduling behavior.

`subscribeOn`, by contrast, controls when the source subscription action itself occurs.

## 7. Placement vs supervision

### delay

```text
value HAS arrived
    -> hold/reposition delivery
```

### timeout

```text
value has NOT arrived
    -> waiting remains valid only until deadline
```

Therefore:

```text
delay   = post-arrival temporal policy
timeout = pre-arrival temporal policy
```

## 8. Temporal competition

`timeout` can be modeled as:

```text
SOURCE ACTIVITY
      versus
DEADLINE
```

If source activity wins, normal observation continues.

If the deadline wins, timeout policy activates.

This is a temporal analogue of competitive policies seen elsewhere in the atlas.

## 9. Measurement vs control

```text
timeout
    time -> CONTROL FLOW

timestamp / timeInterval
    time -> DATA
```

This is one of the most important distinctions in the package.

## 10. Absolute vs relative temporal measurement

### timestamp

```text
v_i -> (v_i, t_i)
```

Question:

```text
WHEN did this value occur?
```

### timeInterval

```text
v_i -> (v_i, t_i - t_(i-1))
```

Question:

```text
HOW LONG since the preceding measurement point?
```

Thus:

```text
timestamp     = temporal coordinate
timeInterval  = temporal delta
```

## 11. Memory topology of measurement

### timestamp

```text
CLOCK ONLY
```

No previous-time history is required.

### timeInterval

```text
CLOCK
+
PREVIOUS-TIME REGISTER
```

The state topology itself explains the difference between absolute and relative measurement.

## 12. Cross-package connections

### delay vs Package 01 temporal filtering

```text
delay
    retains independent pending values

audit / debounce-style policies
    collapse or replace pending values according to admission policy
```

### timeInterval vs Package 04 pairwise

```text
pairwise
    previous VALUE register

timeInterval
    previous TIME register
```

Same broad state topology, different state domain and action policy.

### timeout vs Package 04 race

```text
race
    source vs source competition

timeout
    source activity vs deadline competition
```

## 13. Clock and scheduler are related but distinct

A scheduler can participate in executing work.
A clock/timestamp provider can participate in observing time.

```text
scheduler
    execution placement

clock
    temporal measurement
```

Some RxJS abstractions expose both concerns, but the behavioral roles should remain conceptually separate.

## 14. Business DSL vocabulary

```text
delay          -> deliverAfter / shiftBy
delayWhen      -> releaseWhen
observeOn      -> deliverOn
subscribeOn    -> beginOn
timeout         -> requireWithin
timestamp       -> withTimestamp / annotateTime
timeInterval    -> withElapsedTime
```

## 15. Decision model

```text
Need to move delivery later by a general timing rule?
    -> delay

Need each value to have an independent release signal?
    -> delayWhen

Need downstream notification delivery on a scheduler?
    -> observeOn

Need source subscription scheduled?
    -> subscribeOn

Need excessive waiting to become a control-flow outcome?
    -> timeout

Need absolute occurrence-time metadata?
    -> timestamp

Need elapsed time between emissions?
    -> timeInterval
```

## 16. Package 06 semantic summary

```text
Ask six questions:

1. What temporal subject is controlled or observed?
2. Is the time reference absolute, relative, or event-driven?
3. Is a clock being observed or a scheduler executing work?
4. What temporal state must be retained?
5. What event triggers the temporal policy?
6. Does time MOVE data, CONTROL flow, SCHEDULE work, or become DATA?
```

Package 06 deliberately stops at seven operators because these seven form a coherent temporal algebra without padding the package with deprecated or behaviorally unrelated APIs.
