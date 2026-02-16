# Validation Conventions

> Where to validate and how. See [AGENTS.md](../AGENTS.md) for the layer summary table.

---

## Layer Responsibilities

| Layer | Validates | Examples |
|-------|-----------|----------|
| **Serializer** | Request data format & types | Field types, formats, required fields |
| **View** | Permissions & access control | User authorization, object ownership |
| **Use Case** | Business logic & domain rules | State machine, cross-entity validation |

---

## Serializer Validation

### Field-Level Validation

Use `validate_<field>` methods for specific field rules:

```python
class DriverSerializer(serializers.Serializer):
    email = serializers.EmailField()
    phone = serializers.CharField()

    def validate_email(self, value: str) -> str:
        if "@superdispatch.com" in value:
            raise serializers.ValidationError("Cannot use company email for drivers")
        return value.lower()
```

### Object-Level Validation

Use `validate` method for cross-field validation:

```python
class DateRangeSerializer(serializers.Serializer):
    start_date = serializers.DateField()
    end_date = serializers.DateField()

    def validate(self, attrs: dict) -> dict:
        if attrs["start_date"] > attrs["end_date"]:
            raise serializers.ValidationError("start_date must be before end_date")
        return attrs
```

### Custom Validators

<!-- CUSTOMIZE: Replace path with your project's validators location -->
Reusable validators live in `apps/common/validators.py`:

```python
def validate_vin(value: str) -> str:
    if len(value) != 17:
        raise serializers.ValidationError("VIN must be 17 characters")
    return value.upper()

# Usage
class VehicleSerializer(serializers.Serializer):
    vin = serializers.CharField(validators=[validate_vin])
```

---

## View Validation

### Permission Classes

```python
from rest_framework.permissions import IsAuthenticated
from apps.common.permissions import IsDispatcher, ObjectBelongsToCarrier

class DriverViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated, IsDispatcher, ObjectBelongsToCarrier]
```

### Validation in Views

Prefer `raise_exception=True`:

```python
def post(self, request):
    serializer = DriverSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    # Continue with validated data...
```

---

## Use Case Validation

### Existence Checks

```python
class UpdateDriver:
    def execute(self, driver_guid: str, name: str) -> Driver:
        driver = Driver.objects.filter(guid=driver_guid).first()
        if not driver:
            raise DriverDoesNotExist()
        driver.name = name
        driver.save(update_fields=["name"])
        return driver
```

### Business Rules

```python
class AcceptLoad:
    def execute(self, load: Load, carrier: Carrier) -> Load:
        if load.status != Load.Status.OFFERED:
            raise CannotAcceptLoad("Load must be in OFFERED status")
        if carrier.is_suspended:
            raise CarrierIsSuspended()
        load.status = Load.Status.ACCEPTED
        load.carrier = carrier
        load.save()
        return load
```

### Cross-Entity Validation

```python
class RegisterDriver:
    def execute(self, email: str, carrier: Carrier) -> Driver:
        if User.objects.filter(email=email).exists():
            raise EmailAlreadyExists()
        if Driver.objects.filter(email=email).exists():
            raise EmailAlreadyExists()
        return Driver.objects.create(email=email, carrier=carrier)
```

---

## Summary

| Validation Type | Layer | Example |
|-----------------|-------|---------|
| Field format/type | Serializer | Email format, phone format |
| Required fields | Serializer | Missing required data |
| Cross-field rules | Serializer | Date range validation |
| User authentication | View | IsAuthenticated permission |
| Object ownership | View | ObjectBelongsToCarrier permission |
| Entity existence | Use Case | DriverDoesNotExist |
| State transitions | Use Case | CannotAcceptLoad |
| Business rules | Use Case | CarrierIsSuspended |
| Cross-entity uniqueness | Use Case | EmailAlreadyExists |
