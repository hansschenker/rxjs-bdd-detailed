# <operator>

## 1. Semantic essence

## 2. Classification

## 3. Internal state

## 4. BDD scenarios

## 5. Mealy machine

## 6. Transition table

## 7. Marble diagram

Use lanes appropriate to actual semantics:

```text
source
internal state
notifier / duration / timer / inner streams
sink
```

Lifecycle markers:

```text
^ subscription
| completion
# error
! boundary/notifier signal
x cancellation (ASCII form)
```

## 8. Lifecycle semantics

### Completion

### Error

### Unsubscription / cancellation

## 9. Business use case

## 10. Domain-specific interpretation / DSL

## 11. TypeScript example

## 12. Neighboring-operator comparison

## 13. Semantic fingerprint
