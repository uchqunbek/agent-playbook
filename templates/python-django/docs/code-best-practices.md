# Code Best Practices

> Detailed code conventions with examples. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Readability & Control Flow

Prefer **early returns** to reduce nesting:

```python
# ✅ Good — early returns
def process_order(order):
    if not order:
        raise OrderDoesNotExist()
    if order.is_cancelled:
        raise CannotProcessOrder("Order is cancelled")
    return _do_processing(order)

# ❌ Bad — deeply nested
def process_order(order):
    if order:
        if not order.is_cancelled:
            return _do_processing(order)
        else:
            raise CannotProcessOrder("Order is cancelled")
    else:
        raise OrderDoesNotExist()
```

Keep `if` conditions simple. Extract complex logic into named variables:

```python
# ✅ Good — extracted condition
is_eligible = order.is_active and not order.is_archived and order.has_valid_carrier
if is_eligible:
    process(order)

# ❌ Bad — compound condition
if order.is_active and not order.is_archived and order.has_valid_carrier:
    process(order)
```

Use pythonic boolean expressions — `a != b` over `not a == b`, `and`/`or` over `&`/`|` with booleans.

---

## Imports

Use **absolute imports**:

```python
# ✅ Good
from apps.drivers.use_cases import DeactivateDriver
from apps.common.exceptions import UseCaseError

# ❌ Bad
from .use_cases import DeactivateDriver
from ..common.exceptions import UseCaseError
```

---

## Logging & Exceptions

Log with `logger.exception(...)` to retain tracebacks:

```python
# ✅ Good — includes traceback and context
try:
    process_payment(payment_id)
except PaymentGatewayError as e:
    logger.exception("Payment processing failed", extra={"payment_id": payment_id})
    raise

# ❌ Bad — loses traceback
except PaymentGatewayError as e:
    logger.error(f"Payment failed: {e}")
```

---

## Enums & Choices

<!-- CUSTOMIZE: Replace examples with your project's actual enums -->
Prefer `enum.Enum` for discrete states. Use `models.TextChoices` for Django model fields:

```python
from enum import Enum

class TrialEndAction(Enum):
    CANCEL_SUBSCRIPTION = "cancel_subscription"
    ACTIVATE_SUBSCRIPTION = "activate_subscription"

# Django model choices
class LoadStatus(models.TextChoices):
    PENDING = "pending"
    IN_TRANSIT = "in_transit"
    DELIVERED = "delivered"
```

---

## Database Migrations

- **Never edit auto-generated migration files** — create a new migration instead
- Always use Docker for Django commands: `docker exec -it your-container python manage.py makemigrations`

---

## Messaging & Pub/Sub

Provide dedicated publisher helpers in `apps/{app}/pubsub/publisher.py`:

```python
# ✅ Good — centralized publisher
# apps/orders/pubsub/publisher.py
def publish_order_created(order):
    publish_message_to_rabbitmq_exchange(
        routing_key="order.created",
        payload=build_order_payload(order)
    )

# ❌ Bad — inline publishing in use case
class CreateOrder:
    def execute(self, ...):
        order = Order.objects.create(...)
        publish_message_to_rabbitmq_exchange(...)  # Don't do this inline
```

---

## API Patterns

- **GET by GUID**: Use a use case to raise domain errors (`DriverDoesNotExist`)
- **GET list**: Never raise "not found" — return an empty list
- **Inputs/outputs**: Validate via Serializers, including query params
- **N+1**: Handle with `select_related`/`prefetch_related` in use cases

---

## Celery Conventions

<!-- CUSTOMIZE: Replace queue naming with your project's pattern -->
- Queue naming: `your-project.celery.{domain}`
- Tasks are transport-only — pass identifiers to use cases for DB access
- Retry with specific exceptions and exponential backoff

```python
# ✅ Good — specific retry
@app.task(autoretry_for=(ConnectionError, TimeoutError), retry_backoff=True, max_retries=5)
def sync_payment(payment_guid: str):
    SyncPayment().execute(guid=payment_guid)

# ❌ Bad — blanket retry
@app.task(autoretry_for=(Exception,))
def sync_payment(payment_guid: str):
    ...
```

---

## Model Design

- Derived state as **read-only model properties** (e.g., `is_submitted`, `abbreviation`)
- **Mutations only in use cases** — no model methods that change state

---

## Segment Analytics

<!-- CUSTOMIZE: Remove this section if you don't use Segment -->
Event names MUST use **Title Case with spaces**:

```python
# ✅ Good
segment.track_event(user_guid=user.guid, event_name="Carrier Updated Profile")

# ❌ Bad
segment.track_event(user_guid=user.guid, event_name="carrier_updated_profile")
```
