# Effective Java Mapping Reference

Mapping of "Effective Java" 3rd Edition items to Java Doctor rules. This maps Joshua Bloch's recommendations to specific code checks.

## Item-by-Item Mapping

### Chapter 2: Creating and Destroying Objects

| Item | Title | Related Rules | Description |
|------|-------|---------------|-------------|
| Item 1 | Consider static factory methods instead of constructors | BP-008 | Use static factory instead of/with constructors |
| Item 2 | Builder pattern when faced with many parameters | BP-008, ARCH-003 | Use Builder for objects with many parameters |
| Item 3 | Enforce singleton with private constructor or enum | EJ-009 | Use enum or private constructor |
| Item 4 | Enforce noninstantiability with private constructor | EJ-009 | Utility classes should have private constructor |
| Item 5 | Prefer dependency injection | SPR-007, ARCH-010 | Inject dependencies, don't hardcode |
| Item 6 | Avoid finalizers | RES-001 | Don't use finalizers, use try-with-resources |
| Item 7 | Prefer try-with-resources to try-finally | EJ-011, RES-001 | Always use try-with-resources |

### Chapter 3: Methods Common to All Objects

| Item | Title | Related Rules | Description |
|------|-------|---------------|-------------|
| Item 8 | Override equals properly | EJ-010, BP-005 | If overriding equals, follow contract |
| Item 9 | Always override hashCode when overriding equals | EJ-010 | hashCode and equals must be consistent |
| Item 10 | Always override toString | BP-005 | Provide useful toString output |
| Item 11 | Override clone judiciously | EJ-007 | Be careful with clone, consider copy factories |
| Item 12 | Consider implementing Comparable | EJ-010 | Implement compareTo for sorting |

### Chapter 4: Classes and Interfaces

| Item | Title | Related Rules | Description |
|------|-------|---------------|-------------|
| Item 13 | Minimize accessibility of classes and members | ARCH-010, BP-009 | Make fields private |
| Item 14 | In public classes, use accessor methods | BP-009 | Don't expose public fields |
| Item 15 | Minimize mutability | NULL-008 | Make classes immutable when possible |
| Item 16 | Favor composition over inheritance | ARCH-004 | Don't extend unless is-a relationship |
| Item 17 | Design and document for inheritance or forbid it | ARCH-006 | Document which methods are for override |
| Item 18 | Prefer interfaces to abstract classes | ARCH-006 | Use interfaces for multiple inheritance |
| Item 19 | Use interfaces only to define types | ARCH-006 | Don't use marker interfaces unnecessarily |
| Item 20 | Prefer class hierarchies to tagged classes | ARCH-001 | Don't use switch on type tags |
| Item 21 | Use function objects (strategies) | EJ-022 | Use lambdas for strategies |
| Item 22 | Favor static member classes over nonstatic | EJ-004 | Use static nested classes |

### Chapter 5: Generics

| Item | Title | Related Rules | Description |
|------|-------|---------------|-------------|
| Item 23 | Don't use raw types | EJ-006 | Always use generic types |
| Item 24 | Eliminate unchecked warnings | EJ-006 | Fix raw type warnings |
| Item 25 | Prefer lists to arrays | ARCH-007, PERF-012 | Arrays don't enforce generics |
| Item 26 | Favor generic types | EJ-006 | Make classes generic |
| Item 27 | Favor generic methods | EJ-006 | Make methods generic |
| Item 28 | Use bounded wildcards for flexibility | EJ-006 | Use ? extends, ? super |
| Item 29 | Consider typesafe heterogeneous containers | EJ-006 | Use Map<Class<T>, T> |

### Chapter 6: Enums and Annotations

| Item | Title | Related Rules | Description |
|------|-------|---------------|-------------|
| Item 30 | Use enums instead of int constants | BP-002, EJ-003 | Enums provide type safety |
| Item 31 | Use instance fields instead of ordinal | BP-001 | Don't use ordinal() for storage |
| Item 32 | Use EnumSet instead of bit fields | SUGGESTION | Use EnumSet for flags |
| Item 33 | Use EnumMap instead of ordinal indexing | SUGGESTION | Use EnumMap for enum keys |
| Item 34 | Use interfaces to extend enums | BP-002 | Can implement interfaces on enums |
| Item 35 | Annotations over naming patterns | SUGGESTION | Use annotations, not naming |
| Item 36 | Use @Override consistently | EJ-003 | Always use @Override |
| Item 37 | Marker interfaces define types | SUGGESTION | Use marker interfaces |

### Chapter 7: Lambdas and Streams

| Item | Title | Related Rules | Description |
|------|-------|---------------|-------------|
| Item 38 | Prefer lambdas to anonymous classes | EJ-022 | Use lambdas over anonymous classes |
| Item 39 | Prefer method references | EJ-022 | Use :: method references |
| Item 40 | Prefer standard functional interfaces | EJ-022 | Use java.util.function |
| Item 41 | Use streams judiciously | EJ-020, EJ-021 | Don't overuse streams |
| Item 42 | Use streams with care for side effects | EJ-021 | Don't use streams for side effects |
| Item 43 | Avoid streams when working with char arrays | SUGGESTION | Streams are for objects, not primitives |

### Chapter 8: Methods

| Item | Title | Related Rules | Description |
|------|-------|---------------|-------------|
| Item 44 | Check parameters for validity | NULL-004, EXC-004 | Validate all public method parameters |
| Item 45 | Make defensive copies when needed | NULL-008 | Copy mutable input parameters |
| Item 46 | Design method signatures carefully | API-001, ARCH-003 | Good naming, few parameters |
| Item 47 | Use overloading judiciously | NULL-006 | Be careful with overloaded methods |
| Item 48 | Use varargs judiciously | ARCH-003 | Don't overuse varargs |
| Item 49 | Return empty collections, not nulls | NULL-003 | Never return null from collection methods |
| Item 50 | Return optionals appropriately | EJ-018, EJ-028 | Use Optional<T> as return type |
| Item 51 | Write doc comments for all exposed APIs | BP-005 | Document public APIs |

### Chapter 9: General Programming

| Item | Title | Related Rules | Description |
|------|-------|---------------|-------------|
| Item 52 | Know and use libraries | PERF-012, BP-001 | Use libraries, not custom code |
| Item 53 | Avoid float/double if exact answers needed | BP-001 | Use BigDecimal for money |
| Item 54 | Prefer primitives to boxed primitives | PERF-004 | Don't use Integer when int works |
| Item 55 | Beware string concatenation performance | PERF-003 | Use StringBuilder for loops |
| Item 56 | Interface references over implementation | API-003 | Program to interfaces |
| Item 57 | Use exceptions only for exceptional conditions | EXC-001, EXC-002 | Don't use exceptions for flow |
| Item 58 | Use checked exceptions sparingly | EXC-003 | Don't overuse checked exceptions |
| Item 59 | Favor standard exceptions | EXC-003 | Use IllegalArgumentException, etc. |
| Item 60 | Throw exceptions appropriate to abstraction | EXC-003 | Exception translation |
| Item 61 | Document all exceptions thrown | EXC-003 | Document exceptions in Javadoc |
| Item 62 | Include failure capture information in exceptions | EXC-004 | Include context in exceptions |
| Item 63 | Failure atomicity | SUGGESTION | Keep methods failure-atomic |
| Item 64 | Don't use reflection unnecessarily | SUGGESTION | Reflection is slow and unsafe |
| Item 65 | Prefer native methods sparingly | SUGGESTION | Avoid JNI if possible |
| Item 66 | Optimize judiciously | SUGGESTION | Don't premature optimize |
| Item 67 | Name constants descriptively | BP-001 | Use meaningful constant names |

### Chapter 10: Exceptions

| Item | Title | Related Rules | Description |
|------|-------|---------------|-------------|
| Item 68 | Use exceptions only for exceptional conditions | EXC-001 | Not for control flow |
| Item 69 | Use checked exceptions for recoverable conditions | EXC-003 | Checked for recoverable |
| Item 70 | Use runtime exceptions for programming errors | EXC-003 | Runtime for bugs |
| Item 71 | Avoid unnecessary use of checked exceptions | EXC-003 | Don't overuse checked |
| Item 72 | Prefer standard exceptions to custom | EXC-003 | Use IllegalArgument, etc. |
| Item 73 | Throw exceptions appropriate to abstraction | EXC-003 | Exception translation |
| Item 74 | Document all exceptions thrown | EXC-003 | Javadoc @throws |
| Item 75 | Include failure capture information | EXC-004 | Message, cause |
| Item 76 | Failure atomicity | SUGGESTION | Leave object valid |
| Item 77 | Don't ignore exceptions | EXC-001 | At minimum log |

### Chapter 11: Concurrency

| Item | Title | Related Rules | Description |
|------|-------|---------------|-------------|
| Item 78 | Synchronize access to shared mutable data | CONC-001 | Use synchronized or concurrent |
| Item 79 | Avoid excessive synchronization | CONC-012 | Don't hold lock during I/O |
| Item 80 | Prefer executors to tasks | CONC-008 | Use ExecutorService |
| Item 81 | Prefer concurrency utilities | CONC-010 | Use ConcurrentHashMap, etc. |
| Item 82 | Document thread safety | CONC-001 | @ThreadSafe annotation |
| Item 83 | Use lazy initialization judiciously | CONC-002 | Use holder or double-check |
| Item 84 | Don't depend on thread scheduler | CONC-001 | Don't assume thread timing |
| Item 85 | Prefer nonthreaded singletons | CONC-001 | Enum or holder pattern |
| Item 86 | Don't use thread groups | SUGGESTION | Thread groups are obsolete |

### Chapter 12: Serialization

| Item | Title | Related Rules | Description |
|------|-------|---------------|-------------|
| Item 87 | Consider Serialization carefully | SUGGESTION | It's complex, consider alternatives |
| Item 88 | Write readObject defensively | SEC-008 | Validate in readObject |
| Item 89 | For control, prefer enum to readResolve | SUGGESTION | Enum is simpler |
| Item 90 | Consider serialization proxy pattern | SUGGESTION | Use proxy pattern |

---

## Quick Reference: Effective Java → Rules

```
EJ-001: Item 1 - Static factory methods
EJ-002: Item 2 - Builder pattern  
EJ-003: Item 3, 4 - Singleton/enforce noninstantiability
EJ-004: Item 22 - Static nested classes
EJ-005: Item 5 - Dependency injection
EJ-006: Items 23-29 - Generics
EJ-007: Item 11 - clone() / mutable objects
EJ-008: Item 12 - compareTo() contract
EJ-009: Item 3 - Singleton pattern
EJ-010: Items 8, 9, 10, 12 - Object methods
EJ-011: Item 9 - try-with-resources
EJ-012: Items 45, 26 - var and generics
EJ-013: Item 42 - Lambda type inference
EJ-014: Item 47 - String improvements (Java 11)
EJ-015: Item 51 - Files utility (Java 11)
EJ-016: Item 48 - Immutable collections
EJ-017: Item 48 - List.of(), etc.
EJ-018: Item 44 - Optional usage
EJ-019: Item 45 - Optional.get()
EJ-020: Item 48 - Stream API
EJ-021: Item 48 - Streams side effects
EJ-022: Item 42 - Method references
EJ-023: Item 3 - Sealed classes (Java 17)
EJ-024: Item 3 - Pattern matching (Java 16)
EJ-025: Item 3 - Records (Java 16)
EJ-026: Item 3 - Switch expressions (Java 14)
EJ-027: Item 3 - Text blocks (Java 15)
EJ-028: Item 55 - Optional return
EJ-029: Item 46 - Unnecessary map()
EJ-030: Item 55 - orElseGet()
```

---

## Checklist by Priority

### Critical (Must Fix)

- [ ] **Item 7**: Use try-with-resources (EXC-005, RES-001)
- [ ] **Item 8, 9**: Override equals/hashCode together (EJ-010)
- [ ] **Item 13, 14**: Minimize visibility, use accessors (ARCH-010, BP-009)
- [ ] **Item 23**: Don't use raw types (EJ-006)
- [ ] **Item 52**: Validate method parameters (NULL-004)
- [ ] **Item 53**: Don't use float/double for money (BP-001)
- [ ] **Item 54**: Avoid boxing in loops (PERF-004)
- [ ] **Item 64**: Synchronize shared mutable data (CONC-001)
- [ ] **Item 67**: Use try-with-resources (EJ-011)

### Important (Should Fix)

- [ ] **Item 1**: Static factory methods (BP-008)
- [ ] **Item 2**: Builder for many parameters (BP-008)
- [ ] **Item 5**: Dependency injection (SPR-007)
- [ ] **Item 10**: Override toString (BP-005)
- [ ] **Item 15**: Minimize mutability (NULL-008)
- [ ] **Item 16**: Favor composition (ARCH-004)
- [ ] **Item 19**: Use interfaces for types (ARCH-006)
- [ ] **Item 20**: Class hierarchies over tagged (ARCH-001)
- [ ] **Item 38, 39**: Lambdas and method references (EJ-022)
- [ ] **Item 40**: Use streams judiciously (EJ-020)
- [ ] **Item 44**: Check parameters (NULL-004)
- [ ] **Item 46**: Design signatures (API-001)
- [ ] **Item 47**: Overloading carefully (NULL-006)
- [ ] **Item 49**: Empty collections, not null (NULL-003)
- [ ] **Item 50**: Optional return (EJ-018)
- [ ] **Item 51**: Document exceptions (EXC-003)
- [ ] **Item 55**: StringBuilder for loops (PERF-003)
- [ ] **Item 68**: Exceptions for exceptional cases (EXC-001)
- [ ] **Item 77**: Don't ignore exceptions (EXC-001)
- [ ] **Item 78**: Synchronize shared data (CONC-001)
- [ ] **Item 80**: Prefer executors (CONC-008)
- [ ] **Item 81**: Use concurrent utilities (CONC-010)

### Nice to Have

- [ ] **Item 3**: Singleton/enum (EJ-009)
- [ ] **Item 4**: Private constructor (EJ-009)
- [ ] **Item 6**: Avoid finalizers (RES-001)
- [ ] **Item 11**: clone() carefully (EJ-007)
- [ ] **Item 12**: Comparable (EJ-010)
- [ ] **Item 17**: Document inheritance (ARCH-006)
- [ ] **Item 21**: Function objects (EJ-022)
- [ ] **Item 30**: Enums vs int (BP-002)
- [ ] **Item 36**: @Override always (EJ-003)
- [ ] **Item 41**: Streams carefully (EJ-020)
- [ ] **Item 45**: Defensive copies (NULL-008)
- [ ] **Item 56**: Interface references (API-003)
- [ ] **Item 60**: Exception translation (EXC-003)
- [ ] **Item 61**: Document exceptions (EXC-003)
- [ ] **Item 63**: Failure atomicity (SUGGESTION)
- [ ] **Item 64**: Reflection sparingly (SUGGESTION)
- [ ] **Item 65**: Optimize judiciously (SUGGESTION)
- [ ] **Item 82**: Document thread safety (CONC-001)
- [ ] **Item 84**: Don't depend on scheduler (CONC-001)
