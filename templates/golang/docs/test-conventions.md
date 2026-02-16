# Test Conventions

> Detailed testing rules for Go. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Test Structure

### File Organization

Unit tests live alongside source files. Integration tests in `tests/`:

```
internal/
├── cache/
│   ├── token_cache.go
│   └── token_cache_test.go       # Unit tests (same package)
├── jwt/
│   ├── parse.go
│   └── parse_test.go
├── token/
│   ├── extract.go
│   └── extract_test.go
└── transport/http/token/
    ├── status_handler.go
    └── status_handler_test.go
tests/
└── integration/
    ├── integration_test.go       # End-to-end with testcontainers
    └── ha_test.go                # High availability scenarios
```

### Naming

<!-- CUSTOMIZE: Replace with your project's test naming -->
- **Function pattern:** `TestFunctionName__scenario`
- **Double underscore** separates function from scenario
- **Descriptive scenarios:** `not_found`, `invalid_input`, `concurrent_access`

```go
func TestAdd_and_Lookup(t *testing.T) { ... }
func TestLookup__not_found(t *testing.T) { ... }
func TestLookup__expired_entry(t *testing.T) { ... }
func TestConcurrentAccess(t *testing.T) { ... }
```

---

## Table-Driven Tests

The standard pattern for testing multiple scenarios:

```go
func TestExtractJTI(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    string
        wantErr error
    }{
        {
            name:  "valid token",
            input: makeJWT(map[string]any{"jti": "abc123"}),
            want:  "abc123",
        },
        {
            name:    "missing jti",
            input:   makeJWT(map[string]any{"sub": "user1"}),
            wantErr: ErrMissingJTI,
        },
        {
            name:    "malformed token",
            input:   "not.a.jwt",
            wantErr: ErrInvalidToken,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := ExtractJTI(tt.input)
            if tt.wantErr != nil {
                if !errors.Is(err, tt.wantErr) {
                    t.Errorf("want error %v, got %v", tt.wantErr, err)
                }
                return
            }
            if err != nil {
                t.Fatalf("unexpected error: %v", err)
            }
            if got != tt.want {
                t.Errorf("want %q, got %q", tt.want, got)
            }
        })
    }
}
```

---

## HTTP Handler Tests

<!-- CUSTOMIZE: Replace with your project's handler test patterns -->
Use `httptest` for request/response testing:

```go
func TestGetStatus__returns_active(t *testing.T) {
    cache := cache.NewTokenCache(logger.NewNoop())
    handler := NewHandler(cache, "access_token", logger.NewNoop())

    req := httptest.NewRequest(http.MethodGet, "/v1/token/status", nil)
    req.Header.Set("Authorization", "Bearer "+validToken)
    w := httptest.NewRecorder()

    handler.GetStatus(w, req)

    if w.Code != http.StatusOK {
        t.Errorf("want status 200, got %d", w.Code)
    }

    var resp map[string]any
    json.NewDecoder(w.Body).Decode(&resp)
    if resp["revoked"] != false {
        t.Error("expected token not to be revoked")
    }
}
```

---

## Integration Tests

<!-- CUSTOMIZE: Replace with your project's integration test setup -->
Use `testcontainers-go` for real infrastructure:

```go
func TestMain(m *testing.M) {
    ctx := context.Background()
    container, _ := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
        ContainerRequest: testcontainers.ContainerRequest{
            Image:        "rabbitmq:4-management",
            ExposedPorts: []string{"5552/tcp", "5672/tcp"},
            WaitingFor:   wait.ForListeningPort("5672/tcp"),
        },
        Started: true,
    })
    // Set env vars for tests
    os.Exit(m.Run())
}
```

Run integration tests separately:

```bash
make integration-test         # go test -tags=integration ./tests/integration/ -v -timeout 120s
```

---

## Concurrency Tests

Test thread safety with `sync.WaitGroup`:

```go
func TestConcurrentAccess(t *testing.T) {
    c := cache.NewTokenCache(logger.NewNoop())
    var wg sync.WaitGroup

    // Writers
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            c.Add(fmt.Sprintf("jti-%d", i), RevokedEntry{Cause: "test"})
        }(i)
    }

    // Readers
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            c.Lookup(fmt.Sprintf("jti-%d", i))
        }(i)
    }

    wg.Wait()
}
```

---

## Test Helpers

Keep helpers in test files (not exported):

```go
// makeJWT creates a minimal JWT for testing (no signature verification needed)
func makeJWT(claims map[string]any) string {
    header := base64.RawURLEncoding.EncodeToString([]byte(`{"alg":"none"}`))
    payload, _ := json.Marshal(claims)
    encodedPayload := base64.RawURLEncoding.EncodeToString(payload)
    return header + "." + encodedPayload + "."
}
```

---

## Anti-Patterns

| Anti-Pattern | Correct Approach |
|-------------|-----------------|
| External test frameworks (testify) | Use stdlib `testing` package |
| Shared state between tests | Independent setup per test |
| `t.Fatal()` in goroutines | Use `t.Error()` or channels |
| Hardcoded ports in integration tests | Use `testcontainers` dynamic ports |
| Skipping error checks in tests | Assert every error explicitly |
| `time.Sleep()` for synchronization | Use channels or `sync.WaitGroup` |
