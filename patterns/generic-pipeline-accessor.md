# Pattern: Generic Pipeline Accessor (String-Keyed Configuration)

## Signals

- Single entry-point API uses string keys (e.g. `getX(String key)`)
- Behavior is controlled indirectly via maps/config (e.g. `mSwitchAttrMap`)
- No compile-time type guarantees for values
- Adding new keys is easy, but understanding existing ones is hard
- Special-case logic accumulates in a central pipeline
- Multiple concerns (persistence, sync, side-effects) handled in one place

---

## Heuristic

If a system exposes a **generic, string-keyed pipeline** for accessing structured data or behavior,
→ it is likely trading short-term flexibility for long-term loss of structure.

Prefer explicit modeling when:
- values have types
- behavior differs per key
- side effects exist

---

## Action

Prefer one of:

1. **Strongly-typed properties / APIs**
   - `isDevtoolEnabled(): boolean`
   - `setDevtoolEnabled(boolean)`

2. **Encapsulated setting objects**
   ```java
   class DevtoolSetting {
     boolean value;
     boolean persist;
     boolean syncToNative;
   }
   ```

3. **Hide string keys internally**
   - Keep string-based storage internal
   - Expose typed interfaces to callers

---

## Rationale

- Restores type safety and discoverability
- Localizes behavior instead of scattering it across maps + conditionals
- Prevents growth of a “God object” pipeline
- Makes API the source of truth, not conventions

---

## Trade-offs

- Higher upfront cost to define APIs and types
- Less flexible for rapid prototyping or dynamic keys
- Requires migration effort from generic pipeline

---

## Examples

```java
getDevtoolEnv("enable_feature");   // ❌ implicit, untyped, convention-based

isDevtoolEnabled();               // ✅ explicit, typed
setDevtoolEnabled(true);          // ✅ explicit behavior
```

---

## References

- 2026-03-24__generic-pipeline-accessor.md
