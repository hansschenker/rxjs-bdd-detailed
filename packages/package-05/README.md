# Package 05 - Lifecycle-Directed Stream Composition

Package 05 of **RxJS BDD Detailed** analyzes direct and pipeable stream composition, lifecycle-boundary decoration, terminal coordination, error-insensitive sequencing, and teardown observation.

## Operators

1. `concat`
2. `concatWith`
3. `merge`
4. `mergeWith`
5. `startWith`
6. `endWith`
7. `forkJoin`
8. `onErrorResumeNext`
9. `onErrorResumeNextWith`
10. `finalize`

## Central teaching model

```text
LIFECYCLE-DIRECTED STREAM COMPOSITION
=
PARTICIPANT FORM
+
ACTIVE-SUBSCRIPTION TOPOLOGY
+
NEXT POLICY
+
COMPLETE POLICY
+
ERROR POLICY
+
TEARDOWN POLICY
```

See `synthesis.md` for the package-level analysis.
