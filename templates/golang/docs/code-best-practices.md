# Code Best Practices

> Detailed code conventions for Go. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Package Organization

### Standard Layout

<!-- CUSTOMIZE: Replace with your project's package layout -->

| Directory | Purpose | Importability |
|-----------|---------|---------------|
| `cmd/` | Application entry points | — |
| `internal/` | Private packages | Only within this module |
| `pkg/` | Public reusable packages | Importable by other modules |
| `config/` | Configuration loading | — |
| `tests/` | Integration tests | — |

### Naming

- **Packages**: short, lowercase, no underscores (`cache`, `token`, `http`)
- **Files**: `snake_case.go` (`connection_manager.go`, `status_handler.go`)
- **Test files**: `*_test.go` in same package
- **Interfaces**: named by behavior (`Reader`, `Logger`, `Cache`)

---

## Constructor Pattern

All dependencies injected via `New*()` constructors:

```go
type Handler struct {
    cache      *cache.TokenCache
    logger     logger.Logger
    cookieName string
}

func NewHandler(cache *cache.TokenCache, cookieName string, log logger.Logger) *Handler {
    return &Handler{
        cache:      cache,
        logger:     log,
        cookieName: cookieName,
    }
}
```

<!-- CUSTOMIZE: Replace with your project's DI patterns -->
Wire dependencies in `main.go`:

```go
func main() {
    cfg := config.Load()
    log := logger.New(cfg.LogLevel)

    tokenCache := cache.NewTokenCache(log)
    handler := token.NewHandler(tokenCache, cfg.CookieName, log)

    r := http.NewRouter(handler, log)
    srv := &http.Server{Addr: cfg.HTTPPort, Handler: r}
    // ...
}
```

---

## Interfaces

Define interfaces **where they're consumed**, not where they're implemented:

```go
// pkg/logger/logger.go — interface definition
type Logger interface {
    Info(msg string, args ...any)
    Error(msg string, args ...any)
    Debug(msg string, args ...any)
}

// internal/transport/http/token/status_handler.go — uses Logger interface
type Handler struct {
    logger logger.Logger  // Depends on interface, not concrete type
}
```

---

## Error Handling

### Defined Errors

```go
var (
    ErrInvalidToken = errors.New("invalid token")
    ErrMissingJTI   = errors.New("missing jti claim")
    ErrNotFound     = errors.New("not found")
)
```

### Error Checking

```go
// ✅ Good — use errors.Is()
if errors.Is(err, ErrNotFound) {
    http.WriteError(w, http.NotFoundError("resource not found"))
    return
}

// ✅ Good — wrap errors with context
return fmt.Errorf("failed to parse token: %w", err)

// ❌ Bad — string comparison
if err.Error() == "not found" { ... }

// ❌ Bad — swallow errors silently
result, _ := doSomething()
```

### Handler Error Pattern

```go
func (h *Handler) GetStatus(w http.ResponseWriter, r *http.Request) {
    token, err := extractToken(r)
    if err != nil {
        h.logger.Error("failed to extract token", logger.Error(err))
        pkghttp.WriteJSON(w, http.StatusOK, response)  // Pass-through on error
        return
    }
    // ...
}
```

---

## Concurrency

### Mutex for Shared State

```go
type TokenCache struct {
    mu      sync.RWMutex
    entries map[string]RevokedEntry
    logger  logger.Logger
}

// Read lock for lookups
func (c *TokenCache) Lookup(jti string) (RevokedEntry, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    entry, ok := c.entries[jti]
    return entry, ok
}

// Write lock for mutations
func (c *TokenCache) Add(jti string, entry RevokedEntry) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.entries[jti] = entry
}
```

### Context for Cancellation

```go
func (cm *ConnectionManager) Start(ctx context.Context) {
    cm.once.Do(func() {
        go cm.connectLoop(ctx)
    })
}

func (cm *ConnectionManager) connectLoop(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            return
        default:
            if err := cm.connect(); err != nil {
                time.Sleep(cm.backoff())
                continue
            }
            return
        }
    }
}
```

### Graceful Shutdown

<!-- CUSTOMIZE: Replace with your project's shutdown pattern -->
```go
quit := make(chan os.Signal, 1)
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
<-quit

ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()
srv.Shutdown(ctx)
```

---

## HTTP Response Helpers

<!-- CUSTOMIZE: Replace with your project's response pattern -->
Consistent JSON responses via `pkg/http/`:

```go
// Success response
pkghttp.WriteJSON(w, http.StatusOK, map[string]any{
    "status":  "active",
    "revoked": false,
})

// Error response
pkghttp.WriteError(w, pkghttp.BadRequestError("invalid input"))
```

---

## Logging

<!-- CUSTOMIZE: Replace with your project's logging setup -->
Structured JSON logging via `slog`:

```go
h.logger.Info("token validated",
    logger.String("jti", jti),
    logger.Duration("lookup_time", elapsed),
)

h.logger.Error("cache lookup failed",
    logger.Error(err),
    logger.String("jti", jti),
)
```

---

## Configuration

<!-- CUSTOMIZE: Replace with your project's config pattern -->
Viper with `AutomaticEnv()`:

```go
func Load() Config {
    v := viper.New()
    v.AutomaticEnv()
    v.SetDefault("HTTP_PORT", ":8080")
    v.SetDefault("LOG_LEVEL", "debug")

    return Config{
        HTTPPort: v.GetString("HTTP_PORT"),
        LogLevel: v.GetString("LOG_LEVEL"),
    }
}
```

---

## Anti-Patterns

| Anti-Pattern | Correct Approach |
|-------------|-----------------|
| Global variables for deps | Constructor injection via `New*()` |
| `interface{}` / `any` everywhere | Use typed structs and interfaces |
| Ignoring errors (`_, err`) | Always handle or return errors |
| `panic()` in library code | Return errors; only panic in `main()` |
| String error comparison | Use `errors.Is()` / `errors.As()` |
| Exported fields unnecessarily | Keep struct fields unexported |
| `init()` functions | Explicit initialization in `main()` |
| Shared mutable state without mutex | Use `sync.RWMutex` or channels |
