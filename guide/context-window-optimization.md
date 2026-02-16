# Context Window Optimization

> How to fit maximum useful information into minimum lines.

---

## The 250-Line Rule

`AGENTS.md` should stay **under 250 lines**. Why this number?

| Context Component | Typical Tokens |
|-------------------|---------------|
| System prompt + tool definitions | 5,000-10,000 |
| AGENTS.md (250 lines) | ~3,000 |
| Current file being edited | 2,000-5,000 |
| Search results | 2,000-5,000 |
| Conversation history | 5,000-20,000 |
| **Remaining for output** | **varies** |

Going over 250 lines means every additional line of documentation directly competes with the agent's ability to hold code, search results, and conversation in context.

---

## Condensing Techniques

### Tables Over Prose

```markdown
<!-- ❌ Verbose (8 lines) -->
The project uses several custom annotations. `@JSendResponse` is used on
controller methods to wrap responses in JSend format. `@SpringTransactional`
is our custom transaction annotation that rolls back on all Throwable types.
`@CurrentAuthenticatedUser` extracts the authenticated user from the security
context. `@CheckPermission` enforces permission checks using the
PermissionObject enum.

<!-- ✅ Condensed (6 lines, more scannable) -->
| Annotation | Purpose |
|---|---|
| `@JSendResponse` | JSend-formatted JSON response wrapper |
| `@SpringTransactional` | `@Transactional(rollbackFor = Throwable.class)` |
| `@CurrentAuthenticatedUser` | Extracts authenticated user from security context |
| `@CheckPermission` | Permission enforcement via `PermissionObject` enum |
```

### One-Liners Over Paragraphs

```markdown
<!-- ❌ Verbose -->
When writing code, you should always use the `@RequiredArgsConstructor`
annotation from Lombok combined with `final` fields for dependency injection.
This ensures that all dependencies are injected through the constructor
and are immutable.

<!-- ✅ Condensed -->
- DI via constructor: `@RequiredArgsConstructor` + `final` fields
```

### Links Over Inline Content

```markdown
<!-- ❌ Inline (20+ lines of code examples in AGENTS.md) -->
## Testing Rules
```java
@Test
void method_name_with_condition__expected_result() {
    // Given
    // ... 15 more lines
}
```

<!-- ✅ Link (3 lines in AGENTS.md) -->
## 3. Testing Rules (Key Rules)
- **Naming:** `{method}_{condition(s)}__{expected_result}` — all lowercase, double underscore
- **Full rules with code examples:** [docs/test-conventions.md](docs/test-conventions.md)
```

### ASCII Diagrams Over Descriptions

```markdown
<!-- ❌ Prose -->
The application follows a layered architecture. Controllers receive HTTP
requests and delegate to services. Services contain the business logic and
use repositories for data access. Services also call external API clients
and produce messages to RabbitMQ.

<!-- ✅ Diagram (5 lines, instantly clear) -->
Controller → Service → Repository (JPA)
                     → Client (external APIs)
                     → Message (RabbitMQ producers)
```

---

## The CLAUDE.md Redirect Trick

`CLAUDE.md` is auto-loaded by Claude Code. Instead of duplicating content, make it a 4-line redirect:

```markdown
# CLAUDE.md

All project conventions, architecture, and coding standards are in [AGENTS.md](./AGENTS.md).
This file exists for compatibility with tools that read `CLAUDE.md` (e.g., Claude Code).
```

This saves the full line budget for `AGENTS.md` content and prevents content drift between files.

---

## Real Metrics

### shipper-tms-backend (Java/Spring Boot)

| Metric | Before | After |
|--------|--------|-------|
| AGENTS.md lines | 632 | 235 |
| docs/ files | 0 | 3 |
| Total documentation lines | 632 | 866 |
| Entry point reduction | — | **63%** |
| Content preserved | — | **100%** |

### carrier-monolith (Python/Django)

| Metric | Before | After |
|--------|--------|-------|
| AGENTS.md lines | ~200 | 164 |
| CLAUDE.md lines | ~155 | 4 (redirect) |
| Overlapping content | ~75 lines | 0 |
| Total files | 2 duplicating | 1 source + redirect |
| Net line reduction | — | **-75 lines** |

---

## Checklist

- [ ] `AGENTS.md` is under 250 lines
- [ ] No code examples longer than 5 lines in `AGENTS.md`
- [ ] Every convention in `AGENTS.md` fits on one line
- [ ] Tables used instead of prose for structured information
- [ ] ASCII diagrams used instead of long descriptions
- [ ] `CLAUDE.md` is a 4-line redirect (no duplicated content)
- [ ] All detailed content lives in `docs/` with links from `AGENTS.md`
