# Use Case Conventions

> Detailed use case patterns. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Naming

Do NOT add a `UseCase` suffix:

```python
# ✅ Correct
class StoreLocation:
    pass

# ❌ Wrong
class StoreLocationUseCase:
    pass
```

---

## Single Public Interface

A use case class should only have a single public method named `execute`. All other methods should be `_private`:

```python
class StoreLocation:
    def __init__(self, uow: UnitOfWork):
        self.uow = uow

    async def execute(
        self,
        driver_guid: UUID,
        device_guid: UUID,
        locations_input: LocationListInput,
    ) -> list[Location]:
        locations = self._filter_locations(driver_guid, device_guid, locations_input)
        if not locations:
            return []
        async with self.uow:
            await self.uow.locations.insert_many(locations)

        await send_last_location_to_carrier.kiq(driver_guid)
        return locations

    def _filter_locations(
        self,
        driver_guid: UUID,
        device_guid: UUID,
        locations_input: LocationListInput,
    ) -> list[Location]:
        # Business logic extracted to _private method
        ...
```

---

## Explicit Parameters

Define parameters explicitly — no `**kwargs` or `**data`:

```python
# ✅ Correct
async def execute(self, driver_guid: UUID, device_guid: UUID, data: LocationListInput) -> list[Location]:
    pass

# ❌ Wrong
async def execute(self, **data) -> list[Location]:
    pass
```

---

## No Schemas in Use Cases

Do NOT pass Pydantic schema objects directly to a use case for validation. Pass validated data or model instances:

```python
# ✅ Correct — schema already validated by FastAPI
async def execute(self, driver_guid: UUID, device_guid: UUID, locations_input: LocationListInput):
    locations = self._filter_locations(driver_guid, device_guid, locations_input)
    ...

# ❌ Wrong — use case does schema validation
async def execute(self, raw_data: dict):
    validated = LocationListInput(**raw_data)  # Validation belongs in API layer
```

---

## No Nested Use Cases

Do NOT call a use case inside another use case. If needed, it signals a design problem — refactor the logic.

---

## Unit of Work for Transactions

Multiple database WRITE operations should use the Unit of Work:

```python
class StoreLocation:
    async def execute(self, ...):
        locations = self._filter_locations(...)
        if not locations:
            return []

        async with self.uow:
            await self.uow.locations.insert_many(locations)
        # UoW auto-commits on exiting the context manager
```

---

## Task Dispatch After Commit

Dispatch async tasks (Taskiq `.kiq()`) **after** the UoW context manager exits (after commit), not inside:

```python
# ✅ Correct — dispatch after UoW commit
async with self.uow:
    await self.uow.locations.insert_many(locations)

await send_last_location_to_carrier.kiq(driver_guid)  # After commit

# ❌ Wrong — dispatch inside UoW (data may not be committed yet)
async with self.uow:
    await self.uow.locations.insert_many(locations)
    await send_last_location_to_carrier.kiq(driver_guid)
```

---

## Exception Handling

A use case can only explicitly raise `UseCaseError` subclasses:

```python
class GetLastLocation:
    async def execute(self, driver_guid: UUID) -> Location:
        async with self.uow:
            location = await self.uow.locations.get_latest(driver_guid)
        if not location:
            raise LocationNotFound(driver_guid)
        return location
```

---

## Business Logic Stays in Use Cases

All business logic should be implemented inside the use case. Extract as `_private` methods for readability:

```python
class StoreLocation:
    def _filter_locations(self, ...) -> list[Location]:
        locations_to_insert = []
        now = datetime.now(tz=settings.zoneinfo)

        for location in locations_input.locations:
            is_location_from_future = location.time > now + timedelta(minutes=5)
            is_location_from_past = location.time < now - timedelta(days=31)

            if is_location_from_future or is_location_from_past:
                continue

            locations_to_insert.append(Location(...))

        return locations_to_insert
```

---

## Summary

| Rule | Rationale |
|------|-----------|
| No `UseCase` suffix | Classes describe commands (verbs), not objects (nouns) |
| Single `execute()` | Clear entry point, predictable API |
| Explicit parameters | Self-documenting, type-safe |
| No schemas in use cases | Separation of concerns — views validate, use cases act |
| No nesting | Prevents hidden coupling and tangled dependencies |
| Unit of Work | Transaction consistency for multi-write operations |
| Tasks after commit | Prevents events for rolled-back data |
| UseCaseError only | Consistent error handling across the application |
