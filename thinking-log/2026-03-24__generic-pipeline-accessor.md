# [2026-03-24]

## Context

Refactoring DevTool settings architecture on Android and iOS.

Commits:
https://github.com/lynx-family/lynx/pull/5208/changes/ac6495ab3b1ef644ba6af5167843135e96daae23
https://github.com/lynx-family/lynx/pull/5673/changes/c9e1e7e608d0aecea691c546f65242d37e2418ed

Previous design exposed a generic pipeline:

```java
getDevtoolEnv(String key)
```

Settings were accessed and controlled via string keys, with additional structures (e.g. `mSwitchAttrMap`) to define persistence and native sync behavior.

---

## Observation

The generic “pipeline” approach looks flexible at first, but becomes hard to manage as the system grows.

Symptoms observed:

- Settings are identified by string keys rather than typed interfaces
- Behavior (persist, sync to native, etc.) is controlled indirectly via maps like `mSwitchAttrMap`
- Adding new settings is easy, but understanding existing ones becomes harder
- Special cases start accumulating inside the pipeline

The system starts to feel like:
> everything goes through one entry point, but nothing is clearly modeled

---

## Why

### 1. Flexibility shifts complexity to runtime

The pipeline makes it easy to add new keys:
- no need to define new APIs
- no compile-time constraints

But this removes:
- type safety
- discoverability
- explicit ownership of behavior

As a result:
- correctness depends on conventions (string keys, map entries)
- errors surface later and are harder to trace

---

### 2. Behavior is externalized into configuration

Instead of being encoded in APIs, behavior is scattered in:
- maps (`mSwitchAttrMap`)
- conditional logic inside the pipeline

This creates indirection:
- to understand one setting, you must inspect multiple places
- logic is no longer localized

---

### 3. Tendency toward a “God object”

As soon as some settings have special behavior (e.g. syncing to native C++), the pipeline must handle:

- conditional branching per key
- different side effects
- cross-layer coordination

Over time, the pipeline becomes:
> a central place that knows too much about too many things

This increases:
- complexity
- coupling
- difficulty of change

---

### 4. Loss of API as abstraction boundary

With string-key access:

```java
getDevtoolEnv("some_key")
```

the API no longer communicates:
- what settings exist
- what type they are
- what behavior they carry

The abstraction shifts from:
> “well-defined interface”

to:
> “string + convention”

---

## Decision

Move away from the generic pipeline toward **strongly-typed properties**.

- Each setting is modeled explicitly (e.g. `BOOL` / typed property)
- String keys become internal implementation detail
- Behavior (persist, sync, etc.) is attached to the property itself, not centralized in a pipeline

This makes:
- usage explicit
- types enforceable
- behavior easier to reason about

---

## Confidence

High

This pattern (string-keyed generic pipeline) consistently leads to:
- loss of structure
- accumulation of hidden complexity
- centralization of logic

Trade-off (flexibility vs structure) is clear, and the long-term cost is well understood.
