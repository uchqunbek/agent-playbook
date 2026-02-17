# Test Conventions

> Detailed testing rules for Python + FastAPI. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Test Naming

Use the Super Dispatch naming convention:

```
test__<method>__<expected_result>__given_<condition>
```

```python
# ✅ Good
test__execute__creates_location__given_with_valid_locations
test__execute__not_creates_location__given_with_empty_locations
test__execute__raises_not_found__given_invalid_driver_guid

# ❌ Bad
test_store_location_creates_entries      # Missing convention
test_happy_path                          # Not descriptive
```

---

## Test Structure

### File Organization

Tests mirror the source directory. Each subdirectory has its own `conftest.py`:

```
tests/
├── conftest.py               # Global fixtures (db_session, app, async_client, uow)
├── api/
│   ├── conftest.py           # API-specific fixtures
│   ├── external/
│   │   └── tracking/
│   │       └── test_views.py
│   └── internal/
│       └── tracking/
│           └── test_views.py
├── use_cases/
│   ├── conftest.py           # Domain fixtures (sample_driver_guid, sample_device_guid)
│   ├── test_store_locations.py
│   └── test_get_last_location.py
├── auth/
│   └── conftest.py
└── schemas/
    └── test_rabbit.py
```

---

## Global Fixtures

<!-- CUSTOMIZE: Replace with your project's fixture setup -->
Centralized in `tests/conftest.py`:

```python
_engine = create_async_engine(settings.database_uri, echo=False, pool_pre_ping=True, poolclass=NullPool)
_SessionLocal = sessionmaker(bind=_engine, class_=AsyncSession, expire_on_commit=False)


@pytest.fixture(scope="session", autouse=True)
def event_loop() -> Generator:
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()


@pytest.fixture
async def db_session() -> AsyncGenerator[AsyncSession, None]:
    async with _engine.begin() as conn:
        await conn.execute(text(f"CREATE SCHEMA IF NOT EXISTS {settings.db_schema}"))
        await conn.run_sync(SQLModel.metadata.drop_all)
        await conn.run_sync(SQLModel.metadata.create_all)
        async with _SessionLocal(bind=conn) as session:
            yield session


@pytest.fixture()
def override_get_db_session(db_session: AsyncSession) -> Callable:
    async def _override_get_db():
        yield db_session
    return _override_get_db


@pytest.fixture
def app(override_get_db_session: Callable) -> FastAPI:
    from src.application import app
    from src.db.base import get_db_session
    app.dependency_overrides[get_db_session] = override_get_db_session
    return app


@pytest.fixture
async def async_client(app: FastAPI) -> AsyncGenerator[AsyncClient]:
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as ac:
        yield ac


@pytest.fixture
def client() -> TestClient:
    from src.application import app
    return TestClient(app)


@pytest.fixture(autouse=True, scope="function")
async def clear_global_aiocache() -> None:
    """Clears cache before each test."""
    await cache.clear()


@pytest.fixture()
async def uow(db_session: AsyncSession) -> AsyncGenerator[UnitOfWork, None]:
    async with UnitOfWork(session=db_session) as uow:
        yield uow
```

Key patterns:
- **Session-scoped event loop** for async tests
- **`override_get_db_session`** — helper for dependency override
- **`autouse` cache cleanup** — prevents state leakage between tests
- **UoW fixture uses async context manager** — matches production usage
- **Sync `TestClient`** available for non-async tests

---

## Domain-Specific Fixtures

<!-- CUSTOMIZE: Replace with your project's domain fixtures -->
In `tests/use_cases/conftest.py`:

```python
@pytest.fixture
def sample_driver_guid() -> UUID:
    return UUID("123e4567-e89b-12d3-a456-426614174000")

@pytest.fixture
def sample_device_guid() -> UUID:
    return UUID("123e4567-e89b-12d3-a456-426614174001")

@pytest.fixture
async def last_location(
    db_session: AsyncSession,
    sample_driver_guid: UUID,
    sample_device_guid: UUID,
) -> Location:
    loc = Location(
        latitude=22.22,
        longitude=55.55,
        time=datetime.now(tz=settings.zoneinfo),
        driver_guid=sample_driver_guid,
        device_guid=sample_device_guid,
    )
    db_session.add(loc)
    await db_session.commit()
    await db_session.flush()
    return loc
```

---

## Use Case Tests

<!-- CUSTOMIZE: Replace with your project's test patterns -->
Test business logic with mocked external dependencies:

```python
@pytest.fixture
def mock_send_location_added_message(mocker: MockerFixture) -> MagicMock:
    return mocker.patch("src.use_cases.store_location.send_last_location_to_carrier.kiq")


@pytest.mark.asyncio
async def test__execute__creates_location__given_with_valid_locations(
    mock_send_location_added_message: MagicMock,
    uow: UnitOfWork,
    sample_driver_guid: UUID,
    sample_device_guid: UUID,
):
    now = datetime.now(timezone.utc)
    locations_input = LocationListInput(
        locations=[
            LocationInput(latitude=10.5, longitude=20.3, time=now - timedelta(days=1)),
            LocationInput(latitude=11.5, longitude=21.3, time=now - timedelta(hours=2)),
            LocationInput(latitude=12.5, longitude=22.3, time=now - timedelta(days=60)),
        ]
    )

    result = await StoreLocation(uow).execute(sample_driver_guid, sample_device_guid, locations_input)

    assert len(result) == 2  # Third location filtered (> 31 days old)
    assert await uow.locations.count() == 2
    mock_send_location_added_message.assert_called_once_with(sample_driver_guid)


@pytest.mark.asyncio
async def test__execute__not_creates_location__given_with_empty_locations(
    mock_send_location_added_message: MagicMock,
    uow: UnitOfWork,
    sample_driver_guid: UUID,
    sample_device_guid: UUID,
):
    locations_input = LocationListInput(locations=[])

    result = await StoreLocation(uow).execute(sample_driver_guid, sample_device_guid, locations_input)

    assert len(result) == 0
    mock_send_location_added_message.assert_not_called()
```

---

## Mocking

Use `pytest-mock` for patching. Mock where used, not where implemented:

```python
# ✅ Good — mock at the import location
mocker.patch("src.use_cases.store_location.send_last_location_to_carrier.kiq")

# ❌ Bad — mock at the definition location
mocker.patch("src.tasks.taskiq.send_last_location_to_carrier.kiq")
```

Convention: `mock_` prefix for fixture names, `mocker` as first argument.

---

## Async Test Configuration

In `pyproject.toml`:

```toml
[tool.pytest.ini_options]
asyncio_mode = "auto"
asyncio_default_fixture_loop_scope = "function"
```

With `asyncio_mode = "auto"`, async test functions run without explicit `@pytest.mark.asyncio` (though adding it is also acceptable).

---

## Anti-Patterns

| Anti-Pattern | Correct Approach |
|-------------|-----------------|
| `unittest.TestCase` with async | Use plain `async def test_*` functions |
| All fixtures in one conftest | Domain-specific `conftest.py` per test directory |
| Manual database setup in tests | Use fixtures (`db_session`, `uow`) |
| Testing implementation details | Test behavior via public API |
| Shared mutable state between tests | `autouse` cache cleanup + fresh fixtures per test |
| Hardcoded test data inline | Use fixtures for domain entities |
| Missing mock for async tasks | Always mock `.kiq()` calls in use case tests |
