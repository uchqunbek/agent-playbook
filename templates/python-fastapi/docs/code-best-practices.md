# Code Best Practices

> Detailed code conventions for Python + FastAPI. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Architecture Layers

### Use Cases

<!-- CUSTOMIZE: Replace with your project's use case pattern -->
One class per business operation. Dependencies injected via constructor:

```python
class StoreLocation:
    def __init__(self, uow: UnitOfWork):
        self.uow = uow

    async def execute(
        self,
        user_guid: UUID,
        data: LocationInput,
    ) -> list[Location]:
        locations = [Location(**item.model_dump()) for item in data.items]
        async with self.uow:
            result = await self.uow.locations.insert_many(locations)
        return result
```

### Repositories

<!-- CUSTOMIZE: Replace with your project's repository pattern -->
Generic base with type parameter. Specialized repos extend it:

```python
class BaseRepository(Generic[T]):
    def __init__(self, session: AsyncSession, model: type[T]):
        self.session = session
        self.model = model

    async def insert(self, instance: T) -> T:
        self.session.add(instance)
        await self.session.flush()
        return instance

    async def get_by_id(self, id: int) -> T | None:
        return await self.session.get(self.model, id)


class LocationRepo(BaseRepository[Location]):
    async def get_latest(self, user_guid: UUID) -> Location | None:
        stmt = (
            select(Location)
            .where(Location.user_guid == user_guid)
            .order_by(Location.created_at.desc())
            .limit(1)
        )
        result = await self.session.execute(stmt)
        return result.scalar_one_or_none()
```

### Unit of Work

<!-- CUSTOMIZE: Replace with your project's UoW pattern -->
Wraps database transactions:

```python
async with UnitOfWork(session=session) as uow:
    await uow.locations.insert_many(locations)
    # Auto-commits on success, rolls back on exception
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

async def get_uow(session: AsyncSession = Depends(get_db_session)) -> UnitOfWork:
    return UnitOfWork(session=session)

# In routers
@router.post("/items")
async def create_item(
    data: ItemInput,
    uow: UnitOfWork = Depends(get_uow),
    current_user: User = Depends(get_current_user),
) -> SuccessResponse:
    use_case = CreateItem(uow)
    result = await use_case.execute(current_user.guid, data)
    return SuccessResponse(data=result)
```

---

## Type Hints

<!-- CUSTOMIZE: Adjust based on your mypy strictness -->
mypy enforced with `disallow_untyped_defs=true`:

```python
# ✅ Good — explicit types
async def execute(
    self,
    user_guid: UUID,
    device_guid: UUID,
    data: LocationInput,
) -> list[Location]:
    ...

# ✅ Good — use | for unions (Python 3.10+)
def get_value(key: str) -> str | None:
    ...

# ❌ Bad — Optional syntax
def get_value(key: str) -> Optional[str]:
    ...

# ❌ Bad — missing return type
async def execute(self, user_guid, data):
    ...
```

---

## Error Handling

<!-- CUSTOMIZE: Replace with your project's error patterns -->
Custom exceptions with status codes:

```python
# exceptions.py
class UseCaseError(Exception):
    def __init__(self, message: str, status_code: int = 400):
        self.message = message
        self.status_code = status_code

# In use cases — fail fast with context
if not location:
    raise UseCaseError("Location not found", status_code=404)

# Global exception handler
@app.exception_handler(UseCaseError)
async def use_case_error_handler(request: Request, exc: UseCaseError) -> JSONResponse:
    return FailResponse(message=exc.message, status_code=exc.status_code)
```

---

## Pydantic Schemas

<!-- CUSTOMIZE: Replace with your project's schema patterns -->
Request and response models in `src/schemas/`:

```python
class LocationInput(BaseModel):
    latitude: float
    longitude: float
    timestamp: datetime

    model_config = ConfigDict(
        str_strip_whitespace=True,
        from_attributes=True,
    )

class LocationOutput(BaseModel):
    id: int
    latitude: float
    longitude: float
    created_at: datetime

    model_config = ConfigDict(from_attributes=True)
```

---

## Settings Pattern

<!-- CUSTOMIZE: Replace with your project's settings -->
Pydantic BaseSettings for env-based configuration:

```python
class Settings(BaseSettings):
    db_host: str = "localhost"
    db_port: int = 5432
    db_name: str = "mydb"
    environment: str = "development"

    @property
    def database_uri(self) -> str:
        return f"postgresql+asyncpg://{self.db_user}:{self.db_password}@{self.db_host}:{self.db_port}/{self.db_name}"

settings = Settings()  # Auto-loads from environment
```

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

# ❌ Bad — blocking calls in async code
def get_data():  # Missing async
    result = requests.get(url)  # Blocks event loop
```

---

## Import Style

<!-- CUSTOMIZE: Adjust Ruff isort settings if different -->
Ruff enforces import ordering:

```python
# 1. stdlib
from datetime import datetime
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
| Blocking I/O in async | Use `async` libraries (httpx, asyncpg) |
| Raw SQL strings | Use SQLModel/SQLAlchemy query builder |
| Global mutable state | Use dependency injection |
| `Optional[T]` | Use `T \| None` (Python 3.10+) |
| Relative imports | Use absolute imports (`from src.`) |
| Hardcoded config | Use `Settings` with env vars |
