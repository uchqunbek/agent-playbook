# Naming Conventions

> Detailed naming rules. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Variable Naming

### Local Variables

Use **snake_case** with descriptive compound words:

```python
# ✅ Good
driver_guid = get_driver_guid(request)
filter_params = build_filter_params(request)

# ❌ Bad
driverGuid = get_driver_guid(request)  # camelCase
d = get_driver_guid(request)           # too short
```

### Loop Variables

Use descriptive single words that reflect the item type:

```python
# ✅ Good
for location in locations:
for driver in active_drivers:

# ❌ Bad
for l in locations:
for i in active_drivers:
```

### Boolean Fields and Parameters

Use `is_`, `has_` prefixes:

```python
# ✅ Good
is_location_from_future = location.time > now + timedelta(minutes=5)
is_location_from_past = location.time < now - timedelta(days=31)
has_valid_token = token is not None

# ❌ Bad
location_future = location.time > now     # Missing prefix
futureLocation = location.time > now      # camelCase + missing prefix
```

### Private Methods

Use underscore prefix for non-public methods:

```python
class StoreLocation:
    async def execute(self, driver_guid: UUID, ...):
        locations = self._filter_locations(driver_guid, ...)
        self._validate_input(...)

    def _filter_locations(self, ...) -> list[Location]:
        pass
```

### Constants

Use **ALL_CAPS** with underscores:

```python
GPS_TRACKING_EXCHANGE = "gps_tracking"
DEFAULT_POOL_SIZE = 8
CACHE_TTL_SECONDS = 14400
```

---

## Exception Naming

Exceptions do **NOT** use `Error` or `Exception` suffix. Use domain-specific patterns:

| Pattern | Usage | Examples |
|---------|-------|----------|
| `DoesNotExist` | Entity not found | `LocationNotFound`, `DriverDoesNotExist` |
| `Cannot*` | Action not allowed | `CannotStoreLocation`, `CannotSendOffer` |
| `Already*` | Duplicate state | `AlreadyAccepted`, `AlreadyExists` |
| `Is*` | Invalid state | `IsDeactivated`, `IsLocked` |

### Exception Structure

<!-- CUSTOMIZE: Replace import paths with your project's exception base class -->
```python
from fastapi import status
from src.use_cases.exceptions import UseCaseError

class LocationNotFound(UseCaseError):
    def __init__(self, driver_guid: UUID):
        message = f"Location for driver_guid {driver_guid} not found"
        super().__init__(message, status_code=status.HTTP_404_NOT_FOUND)
```

### Exception Location

Store exceptions in per-domain `exceptions.py` files:

```
src/
  use_cases/
    exceptions.py           # UseCaseError base, LocationNotFound
```

---

## Common Abbreviations

<!-- CUSTOMIZE: Replace with your project's standard abbreviations -->
| Abbreviation | Meaning |
|--------------|---------|
| `guid` | Global Unique Identifier (preferred over `uuid`) |
| `uow` | Unit of Work |
| `loc` | Location (in fixture names) |

---

## Type Hints

Use Python 3.10+ union syntax for all code:

```python
# ✅ Preferred
def get_location(guid: UUID) -> Location | None:
    pass

# ❌ Avoid
def get_location(guid: UUID) -> Optional[Location]:
    pass
```
