# [2026-03-24]

## Context

Initial DevTool configuration used string-based keys (e.g. `enable_devtool`) to represent both:

- persistent user preferences
- transient runtime states

All accessed via a generic pipeline.

## Observation

A single key (e.g. `enable_logbox`) was overloaded to represent:

- what the user wants
- what the system is currently able to do

Example:

```java
getDevtoolEnv("enable_logbox") == false
```

This could mean:

- user disabled it
- OR system is not ready (e.g. engine not initialized)

The caller cannot distinguish between these cases.

## Why

### 1. Loss of information

Two independent variables are collapsed into one value:

- Preference (user intent)
- System State (runtime condition)

Once combined:

- original intent is no longer recoverable
- different causes produce the same result

This makes the system:

> ambiguous and irreversible

### 2. Overloaded representation (string key as channel)

The problem is amplified by using a generic string-keyed map:

- no structure to enforce meaning
- no type system to separate concepts
- easy to accidentally mix unrelated concerns

The key becomes:

> a shared channel carrying multiple semantics

### 3. Mismatch in nature of data

The two concepts have fundamentally different properties:

| Concept      | Nature                   |
| ------------ | ------------------------ |
| Preference   | persistent, user-driven  |
| System State | transient, system-driven |

Combining them forces:

- one to behave like the other
- or both to lose their identity

### 4. Unpredictable system behavior

If system state overrides preference:

- system may “forget” user intent
- behavior depends on timing / lifecycle
- debugging becomes difficult

Because:

> same input → different meaning depending on context

## Decision

- Separate the two sources of truth:
  1. **Preference → DevToolSettings** (what user wants)
  2. **System State → DevToolLifecycle** (what system can do)
- Compute effective behavior explicitly:

```
EffectiveState = f(Preference, SystemState)
```

- Avoid using generic string-keyed pipelines for representing mixed concerns
- Move toward typed properties to enforce separation

## Confidence

High

The “loss of information” problem is fundamental:

- once two dimensions are collapsed into one,
- correctness and debuggability degrade rapidly
