# Code Best Practices

> Detailed code conventions for Python + FastAPI. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Architecture Layers

### Routers vs Views

<!-- CUSTOMIZE: Replace with your project's router/view pattern -->
Separate route registration from handler logic:

```python
# api/internal/tracking/routers.py — route declarations
from src.api.internal.tracking.views import last_location_of_driver
from src.api.responses import SuccessResponse
from src.schemas.locations import LocationSchema

tracking_router = APIRouter()
tracking_router.add_api_route(
    "/{driver_guid}",
    last_location_of_driver,
    methods=["GET"],
    response_model=LocationSchema,
    response_class=SuccessResponse,
)

# api/internal/tracking/views.py — handler logic
async def last_location_of_driver(
    driver_guid: UUID,
    uow: UnitOfWork = Depends(get_uow),
) -> Location:
    location: Location | None = await GetLastLocation(uow).execute(driver_guid)
    if not location:
        raise LocationNotFound(driver_guid)
    return location
```

### Repositories

<!-- CUSTOMIZE: Replace with your project's repository pattern -->
Generic base with type parameter. Specialized repos extend it:

```python
class BaseRepository(Generic[T]):
    def __init__(self, session: AsyncSession, model: Type[T]):
        self.session = session
        self.model = model

    async def insert(self, instance: T) -> T:
        self.session.add(instance)
        await self.session.flush()
        await self.session.refresh(instance)  # Required for SQLModel
        return instance

    async def insert_many(self, instances: list[T]) -> list[T]:
        self.session.add_all(instances)
        await self.session.flush()
        return instances

    async def get_by_id(self, id: int) -> T | None:
        result = await self.session.get(self.model, id)
        return cast(T | None, result)
```

Specialized repos add domain-specific queries:

```python
class LocationRepo(BaseRepository[Location]):
    async def filter(self, **filters: dict) -> list[Location]:
        statement = self._get_filter_statement(**filters)
        result = await self.session.exec(statement)
        return list(result.all())

    async def count(self, **filters: dict) -> int:
        statement = self._get_filter_statement(**filters)
        count_statement = select(func.count()).select_from(statement.subquery())
        result = await self.session.exec(count_statement)
        return cast(int, result.one())
```

### Unit of Work

<!-- CUSTOMIZE: Replace with your project's UoW pattern -->
Wraps database transactions. Auto-commits on success, rolls back on exception:

```python
async with UnitOfWork(session=session) as uow:
    await uow.locations.insert_many(locations)
```

---

## Dependency Injection

<!-- CUSTOMIZE: Replace with your project's DI setup -->
Use FastAPI's `Depends()` for all injectable dependencies:

```python
# dependencies.py
async def get_db_session() -> AsyncGenerator[AsyncSession, None]:
    async with async_session_maker() as session:
        yield session

async def get_uow(session: AsyncSession = Depends(get_db_session)) -> AsyncGenerator[UnitOfWork, None]:
    async with UnitOfWork(session=session) as uow:
        yield uow

async def _get_api_client() -> AsyncGenerator[AsyncClient, None]:
    async with AsyncClient() as client:
        yield client

def verify_internal_token(Authorization: str = Header(None)) -> bool:
    """Service-to-service token verification."""
    if Authorization != settings.carrier_service_token:
        raise HTTPException(status_code=401, detail="Invalid token")
    return True
```

---

## JSend Response Format

<!-- CUSTOMIZE: Replace if your project uses a different response format -->
Three response types following JSend standard:

```python
class SuccessResponse(JSONResponse):
    # {"status": "success", "data": {...}}

class FailResponse(JSONResponse):
    # {"status": "fail", "data": {"message": "...", "details": [...]}}

class ErrorResponse(JSONResponse):
    # {"status": "error", "message": "..."}
```

Use `SuccessResponse` for 2xx, `FailResponse` for 4xx (client errors), `ErrorResponse` for 5xx (server errors).

---

## Exception Handling

<!-- CUSTOMIZE: Replace with your project's exception patterns -->
Base exception with status code and details:

```python
class UseCaseError(Exception):
    def __init__(self, message: str, details: list = [], status_code: int = status.HTTP_400_BAD_REQUEST):
        self.message = message
        self.details = details
        self.status_code = status_code

class LocationNotFound(UseCaseError):
    def __init__(self, driver_guid: UUID):
        message = f"Location for driver_guid {driver_guid} not found"
        super().__init__(message, status_code=status.HTTP_404_NOT_FOUND)
```

Register handlers in the app factory:

```python
# exception_handlers.py
def register_exception_handlers(app: FastAPI) -> None:
    app.add_exception_handler(UseCaseError, use_case_error_handler)

# application.py
register_exception_handlers(app)
```

---

## Settings Pattern

<!-- CUSTOMIZE: Replace with your project's settings -->
Pydantic BaseSettings with Enum-based environment and TestSettings subclass:

```python
class Environment(str, Enum):
    test = "test"
    development = "development"
    staging = "staging"
    production = "production"

class Settings(BaseSettings):
    environment: Environment = Environment.staging
    db_host: str = "localhost"
    db_port: int = 5432
    db_schema: str = "tracking"
    db_pool_max_size: int = 8
    db_pool_max_overflow: int = 4
    db_pool_recycle: int = 3600

    @property
    def database_uri(self) -> str:
        return f"postgresql+asyncpg://{self.db_user}:{self.db_password}@{self.db_host}:{self.db_port}/{self.db_name}"

class TestSettings(Settings):
    db_schema: str = "test"

def get_settings() -> Settings:
    env = os.environ.get("ENVIRONMENT", "staging")
    return TestSettings() if env == "test" else Settings()

settings = get_settings()
```

---

## Type Hints

mypy enforced with `disallow_untyped_defs=true`. Use `cast()` for ORM results:

```python
# ✅ Good — explicit types with | syntax
async def execute(self, driver_guid: UUID, data: LocationInput) -> list[Location]:
    ...

# ✅ Good — cast() for ORM results
result = await self.session.exec(count_statement)
return cast(int, result.one())

# ❌ Bad — Optional syntax
def get_value(key: str) -> Optional[str]:
    ...

# ❌ Bad — missing return type
async def execute(self, driver_guid, data):
    ...
```

---

## SQLModel Definitions

<!-- CUSTOMIZE: Replace with your project's model patterns -->
```python
class Location(BaseModel, table=True):
    id: Optional[int] = Field(
        default=None,
        sa_column=Column(BigInteger(), primary_key=True, autoincrement=True),
    )
    guid: UUID = Field(default_factory=uuid7, unique=True, nullable=False)
    driver_guid: UUID = Field(nullable=False, index=True)
    latitude: float
    longitude: float
    time: datetime | None = Field(default=None, sa_column=Column(DateTime(timezone=True)))
    created_at: datetime = Field(
        sa_column=Column(DateTime(timezone=True)),
        default_factory=lambda: datetime.now(timezone.utc),
    )
```

Key patterns:
- `BigInteger` for IDs on high-volume tables
- `uuid7` for GUIDs (time-ordered)
- Timezone-aware `DateTime` columns
- `index=True` on frequently queried fields

---

## Async Patterns

```python
# ✅ Good — async context managers
async with async_session_maker() as session:
    result = await session.execute(stmt)

# ✅ Good — async generators for DI
async def get_session() -> AsyncGenerator[AsyncSession, None]:
    async with async_session_maker() as session:
        yield session

# ✅ Good — task dispatch after UoW commit
async with self.uow:
    await self.uow.locations.insert_many(locations)
await send_last_location_to_carrier.kiq(driver_guid)  # After commit

# ❌ Bad — blocking calls in async code
result = requests.get(url)  # Blocks event loop
```

---

## Import Style

Ruff enforces import ordering (line length 120):

```python
# 1. stdlib
from datetime import datetime, timedelta
from typing import cast
from uuid import UUID

# 2. third-party
from fastapi import Depends, Request
from pydantic import BaseModel
from sqlmodel import Field, SQLModel

# 3. first-party (absolute imports)
from src.db.models import Location
from src.config.settings import settings
```

---

## Anti-Patterns

| Anti-Pattern | Correct Approach |
|-------------|-----------------|
| `Any` type annotations | Use specific Pydantic models or typed dicts |
| `Optional[T]` | Use `T \| None` (Python 3.10+) |
| Blocking I/O in async | Use `async` libraries (httpx, asyncpg) |
| Raw SQL strings | Use SQLModel/SQLAlchemy query builder |
| Global mutable state | Use dependency injection via `Depends()` |
| Relative imports | Use absolute imports (`from src.`) |
| Hardcoded config | Use `Settings` with env vars |
| Inline route decorators | Use `add_api_route()` in separate `routers.py` |
| Missing `refresh()` after insert | Always `await session.refresh(instance)` |
