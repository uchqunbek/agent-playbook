# Use Case Conventions

> Detailed use case patterns. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Naming

Do NOT add a `UseCase` suffix:

```python
# ✅ Correct
class RegisterUser:
    pass

# ❌ Wrong
class RegisterUserUseCase:
    pass
```

---

## Single Public Interface

A use case class should only have a single public method named `execute`. All other methods should be `_private`:

```python
class RegisterUser:
    def execute(self, carrier: str):
        self._do_something()

    def _do_something(self):
        pass
```

---

## Explicit Parameters

Define parameters explicitly — no `**kwargs` or `**data`:

```python
# ✅ Correct
def execute(self, name: str, email: str, age: int) -> dict:
    pass

# ❌ Wrong
def execute(self, **data) -> dict:
    pass
```

---

## No Serializers in Use Cases

Do NOT pass Django Rest Framework serializer objects to a use case. Pass model instances or primitives:

```python
# ✅ Correct — explicit parameters after view validates
def execute(self, carrier: Carrier, name: str, usdot: str):
    carrier.name = name
    carrier.usdot = usdot
    carrier.save(update_fields=["name", "usdot"])

# ❌ Wrong — serializer passed to use case
def execute(self, carrier: Carrier, data: dict):
    serializer = CarrierSerializer(instance=carrier, data=data, partial=True)
    serializer.is_valid(raise_exception=True)
    serializer.save()
```

---

## No Nested Use Cases

Do NOT call a use case inside another use case. If needed, it signals a design problem — refactor the logic.

---

## Atomic Transactions

Multiple database WRITE operations should be executed under an atomic transaction:

```python
class RegisterUser:
    def execute(self, django_model_instance: DjangoModel):
        with transaction.atomic():
            django_model_instance.field = "value"
            django_model_instance.save()
            self._create_related_entity()

    def _create_related_entity(self):
        AnotherModel.objects.create(...)
```

---

## Signals After Commit

Raise events via Django signals **outside** the atomic transaction, after commit:

```python
class RegisterUser:
    def execute(self, carrier: CarrierModel, email: str):
        with transaction.atomic():
            user = self._add_user(email=email)
            carrier.company_type = "fleet"
            carrier.save(update_fields=["company_type"])

        user_registered.send_robust(sender=None, user=user)
```

---

## Exception Handling

A use case can only explicitly raise `UseCaseError` subclasses. Catch ORM exceptions and re-raise as `UseCaseError`:

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

---

## Business Logic Stays in Use Cases

All business logic should be implemented inside the use case. Do not extract business logic as standalone functions. Extract as `_private` methods of the use case if needed for readability.

---

## Summary

| Rule | Rationale |
|------|-----------|
| No `UseCase` suffix | Classes describe commands (verbs), not objects (nouns) |
| Single `execute()` | Clear entry point, predictable API |
| Explicit parameters | Self-documenting, type-safe |
| No serializers | Separation of concerns — views validate, use cases act |
| No nesting | Prevents hidden coupling and tangled dependencies |
| Atomic transactions | Data consistency for multi-write operations |
| Signals after commit | Prevents events for rolled-back data |
| UseCaseError only | Consistent error handling across the application |
