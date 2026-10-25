# Package 01 - Temporal Observation and Admission Policies

This package is the first reference package of **RxJS BDD Detailed**.

It develops ten RxJS operators through a common behavioral-analysis format:

1. semantic essence
2. classification
3. internal state
4. BDD scenarios
5. Mealy machine
6. transition model
7. marble diagram
8. lifecycle semantics
9. business use case
10. domain-specific interpretation / DSL
11. neighboring-operator comparison
12. semantic fingerprint

## Operators

1. `audit`
2. `auditTime`
3. `debounce`
4. `debounceTime`
5. `throttle`
6. `throttleTime`
7. `sample`
8. `sampleTime`
9. `distinct`
10. `distinctUntilChanged`

## Visual grammar

```text
source
internal state
notifier / duration / timer
sink

^ subscription
| completion
# error
! notifier / duration / timer signal
x cancellation (ASCII form of the cancellation marker)
```

See `synthesis.md` for the package-level semantic model.
