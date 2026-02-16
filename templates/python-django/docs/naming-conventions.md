# Naming Conventions

> Detailed naming rules. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Variable Naming

### Local Variables

Use **snake_case** with descriptive compound words:

```python
# ✅ Good
invoice_total = calculate_total(items)
filter_params = build_filter_params(request)

# ❌ Bad
invoiceTotal = calculate_total(items)  # camelCase
t = calculate_total(items)             # too short
```

### Loop Variables

Use descriptive single words that reflect the item type:

```python
# ✅ Good
for vehicle in vehicles:
for shipment in active_shipments:

# ❌ Bad
for v in vehicles:
for i in active_shipments:
```

### Boolean Fields and Parameters

Use `is_`, `has_`, or `enabled_` prefixes:

```python
# ✅ Good
is_pending_reminder = True
has_login_block = False
enabled_auto_dispatch = carrier.auto_dispatch_enabled

# ❌ Bad
pending_reminder = True    # Missing prefix
loginBlock = False         # camelCase + missing prefix
```

### Private Methods

Use underscore prefix for non-public methods:

```python
class ProcessPayment:
    def execute(self, payment_id: str):
        self._validate_payment(payment_id)
        self._do_something()

    def _validate_payment(self, payment_id: str):
        pass
```

### Constants

Use **ALL_CAPS** with underscores:

```python
SIGNED_URL_EXPIRATION_MINUTES = 15
MAX_SENDGRID_FILE_SIZE = 30 * 1024 * 1024
DEFAULT_PAGE_SIZE = 25
```

---

## Exception Naming

Exceptions do **NOT** use `Error` or `Exception` suffix. Use domain-specific patterns:

| Pattern | Usage | Examples |
|---------|-------|----------|
| `DoesNotExist` | Entity not found | `DriverDoesNotExist`, `LoadDoesNotExist` |
| `Cannot*` | Action not allowed | `CannotEditLoad`, `CannotSendInvoice` |
| `Already*` | Duplicate state | `AlreadyAccepted`, `AlreadyExists` |
| `Is*` | Invalid state | `IsDeactivated`, `IsLocked` |

### Exception Structure

<!-- CUSTOMIZE: Replace import paths with your project's exception base class -->
```python
from rest_framework import status
from apps.common.exceptions import UseCaseError

class DriverDoesNotExist(UseCaseError):
    _code = status.HTTP_404_NOT_FOUND
    _dev_message = "Driver with provided GUID does not exist"
    _user_message = "Driver not found"
```

### Exception Location

Store exceptions in per-app `exceptions.py` files:

```
apps/
  drivers/
    use_cases/
      exceptions.py    # DriverDoesNotExist, CannotDeactivateDriver
  orders/
    use_cases/
      exceptions.py    # LoadDoesNotExist, CannotEditLoad
```

---

## Common Abbreviations

<!-- CUSTOMIZE: Replace with your project's standard abbreviations -->
| Abbreviation | Meaning |
|--------------|---------|
| `guid` | Global Unique Identifier (preferred over `uuid`) |
| `vin` | Vehicle Identification Number |
| `usdot` | US Department of Transportation number |

---

## Type Hints

Use Python 3.10+ union syntax for new code:

```python
# Preferred
def get_driver(guid: str) -> Driver | None:
    pass

# Also acceptable
def get_carrier(guid: str) -> Union[Carrier, None]:
    pass
```
