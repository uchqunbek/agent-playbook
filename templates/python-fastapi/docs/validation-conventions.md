# Validation Conventions

> Where to validate and how. See [AGENTS.md](../AGENTS.md) for the layer summary table.

---

## Layer Responsibilities

| Layer | Validates | Examples |
|-------|-----------|----------|
| **Pydantic Schema** | Request data format & types | Field types, formats, required fields |
| **View/Dependency** | Permissions & authentication | `Depends(get_current_driver)`, token verification |
| **Use Case** | Business logic & domain rules | Time range filtering, entity existence |

---

## Pydantic Schema Validation

### Field-Level Validation

Use Pydantic validators for field rules:

```python
class LocationInput(BaseModel):
    latitude: float
    longitude: float
    timestamp: datetime

    model_config = ConfigDict(
        str_strip_whitespace=True,
        from_attributes=True,
    )

class LocationListInput(BaseModel):
    locations: list[LocationInput]
```

### Custom Validators

<!-- CUSTOMIZE: Replace with your project's validator patterns -->
```python
from pydantic import field_validator

class LocationInput(BaseModel):
    latitude: float
    longitude: float

    @field_validator("latitude")
    @classmethod
    def validate_latitude(cls, v: float) -> float:
        if not -90 <= v <= 90:
            raise ValueError("Latitude must be between -90 and 90")
        return v
```

---

## View/Dependency Validation

### Authentication Dependencies

```python
from fastapi import Depends, Header

async def get_current_driver(
    request: Request,
    api_client: AsyncClient = Depends(_get_api_client),
) -> DriverAuthResponse:
    """Authenticate driver using Token or Bearer JWT."""
    scheme, _ = get_authorization_scheme_param(request.headers.get("Authorization"))
    if scheme_lower == "token":
        return await driver_auth_backend(request, api_client)
    elif scheme_lower == "bearer":
        return await jwt_auth_backend(request)
    raise HTTPException(status_code=401, detail="Unsupported auth scheme")

def verify_internal_token(Authorization: str = Header(None)) -> bool:
    """Service-to-service token verification."""
    if Authorization != settings.carrier_service_token:
        raise HTTPException(status_code=401)
    return True
```

---

## Use Case Validation

### Existence Checks

```python
class GetLastLocation:
    async def execute(self, driver_guid: UUID) -> Location:
        async with self.uow:
            location = await self.uow.locations.get_latest(driver_guid)
        if not location:
            raise LocationNotFound(driver_guid)
        return location
```

### Business Rules

```python
class StoreLocation:
    def _filter_locations(self, ...) -> list[Location]:
        for location in locations_input.locations:
            is_location_from_future = location.time > now + timedelta(minutes=5)
            is_location_from_past = location.time < now - timedelta(days=31)

            if is_location_from_future or is_location_from_past:
                continue  # Skip invalid locations

            locations_to_insert.append(Location(...))
```

---

## Summary

| Validation Type | Layer | Example |
|-----------------|-------|---------|
| Field format/type | Pydantic Schema | Latitude range, email format |
| Required fields | Pydantic Schema | Missing required data |
| Cross-field rules | Pydantic Schema | Date range validation |
| User authentication | Dependency | `Depends(get_current_driver)` |
| Service token | Dependency | `verify_internal_token` |
| Entity existence | Use Case | `LocationNotFound` |
| Business rules | Use Case | Time range filtering |
