# Test Conventions

> Detailed testing rules for Python + FastAPI. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Test Structure

### File Organization

Tests mirror the source directory:

```
tests/
├── conftest.py               # Global fixtures (db_session, app, async_client)
├── api/
│   ├── conftest.py           # API-specific fixtures
│   ├── external/             # External endpoint tests
│   └── internal/             # Internal endpoint tests
├── use_cases/
│   └── conftest.py           # Use case fixtures
├── auth/                     # Auth tests
└── schemas/                  # Schema validation tests
```

### Rules

<!-- CUSTOMIZE: Replace with your project's test conventions -->
- **One test file per module** — mirrors `src/` structure
- **Descriptive test names** — `test_store_location_creates_entries`
- **Async tests** — use `@pytest.mark.asyncio` (or `asyncio_mode = "auto"`)
- **Fixtures for setup** — no manual object construction in test body
- **Clean state** — each test starts with a fresh database

---

## Fixtures

### Core Fixtures

<!-- CUSTOMIZE: Replace with your project's fixture setup -->
Centralized in `tests/conftest.py`:

```python
@pytest.fixture
async def db_session() -> AsyncGenerator[AsyncSession, None]:
    """Create a test database session with fresh tables."""
    engine = create_async_engine(settings.test_database_uri, poolclass=NullPool)
    async with engine.begin() as conn:
        await conn.run_sync(SQLModel.metadata.drop_all)
        await conn.run_sync(SQLModel.metadata.create_all)
    async with AsyncSession(engine) as session:
        yield session
    await engine.dispose()

@pytest.fixture
def app(db_session: AsyncSession) -> FastAPI:
    """Create test app with overridden dependencies."""
    app = create_application()
    app.dependency_overrides[get_db_session] = lambda: db_session
    return app

@pytest.fixture
async def async_client(app: FastAPI) -> AsyncGenerator[AsyncClient, None]:
    """HTTPX async client for testing endpoints."""
    async with AsyncClient(
        transport=ASGITransport(app=app),
        base_url="http://test",
    ) as client:
        yield client

@pytest.fixture
async def uow(db_session: AsyncSession) -> UnitOfWork:
    """Unit of Work with test database."""
    return UnitOfWork(session=db_session)
```

---

## API Tests

<!-- CUSTOMIZE: Replace with your project's API test patterns -->
Test endpoints via `async_client`:

```python
@pytest.mark.asyncio
async def test_create_item_returns_201(
    async_client: AsyncClient,
    auth_headers: dict[str, str],
) -> None:
    response = await async_client.post(
        "/api/v1/items",
        json={"name": "Test Item", "value": 42},
        headers=auth_headers,
    )
    assert response.status_code == 201
    data = response.json()["data"]
    assert data["name"] == "Test Item"


@pytest.mark.asyncio
async def test_create_item_rejects_invalid_input(
    async_client: AsyncClient,
    auth_headers: dict[str, str],
) -> None:
    response = await async_client.post(
        "/api/v1/items",
        json={"name": ""},  # Missing required field
        headers=auth_headers,
    )
    assert response.status_code == 422
```

---

## Use Case Tests

<!-- CUSTOMIZE: Replace with your project's use case test patterns -->
Test business logic with mocked dependencies:

```python
@pytest.mark.asyncio
async def test_store_location_creates_entries(
    uow: UnitOfWork,
    mock_send_notification: MagicMock,
) -> None:
    use_case = StoreLocation(uow)
    result = await use_case.execute(
        user_guid=sample_user_guid,
        data=LocationInput(latitude=40.7, longitude=-74.0, timestamp=now),
    )
    assert len(result) == 1
    assert result[0].latitude == 40.7
    mock_send_notification.assert_called_once()
```

---

## Mocking

<!-- CUSTOMIZE: Replace with your project's mocking patterns -->
Use `pytest-mock` for patching:

```python
@pytest.fixture
def mock_send_notification(mocker: MockerFixture) -> MagicMock:
    """Mock async task dispatch."""
    return mocker.patch("src.use_cases.store_location.send_notification.kiq")

@pytest.fixture
def mock_cache(mocker: MockerFixture) -> MagicMock:
    """Mock cache service."""
    return mocker.patch("src.services.cache.get", return_value=None)
```

---

## Schema Tests

Validate Pydantic models:

```python
def test_location_input_validates_coordinates() -> None:
    data = LocationInput(latitude=40.7, longitude=-74.0, timestamp=datetime.now())
    assert data.latitude == 40.7

def test_location_input_rejects_invalid_latitude() -> None:
    with pytest.raises(ValidationError):
        LocationInput(latitude=999, longitude=-74.0, timestamp=datetime.now())
```

---

## Async Test Configuration

<!-- CUSTOMIZE: Adjust based on your pytest-asyncio version -->
In `pyproject.toml`:

```toml
[tool.pytest.ini_options]
asyncio_mode = "auto"
```

With `asyncio_mode = "auto"`, all async test functions run without explicit `@pytest.mark.asyncio`.

---

## Anti-Patterns

| Anti-Pattern | Correct Approach |
|-------------|-----------------|
| `unittest.TestCase` with async | Use plain `async def test_*` functions |
| Manual database setup in tests | Use fixtures (`db_session`, `uow`) |
| Testing implementation details | Test behavior via public API |
| Shared mutable state between tests | Fresh fixtures per test |
| `time.sleep()` in tests | Use async assertions or `asyncio.wait_for()` |
| Hardcoded test data | Use factories or parametrized fixtures |
