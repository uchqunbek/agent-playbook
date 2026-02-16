# Test Conventions

> Detailed testing rules. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Framework

- **Use pytest** for all new tests
- **Rewrite old tests** to pytest when fixing/changing them

---

## Test Naming

**Pattern:** `test__<method_or_http_method>__<expected_result>__given_<condition>`

```python
# API view tests
def test__post__raises_validation_error__given_invalid_email(): ...
def test__get__returns_200__given_authenticated_user(): ...
def test__delete__returns_404__given_nonexistent_resource(): ...

# Method/function tests
def test__execute__returns_carrier_data__given_valid_usdot(): ...
def test__validate__raises_error__given_invalid_input(): ...
```

**Tips:**
- Use `given` or `when` instead of `if`
- Omit `_given_` if it's obvious from context
- Always use double-underscore `test__<method>__...`

---

## Test Structure (Given-When-Then)

```python
def test__example(mocker: MockerFixture):
    # Given
    mock_service = mocker.patch('path.to.service')
    test_data = {"key": "value"}

    # When
    result = function_under_test(test_data)

    # Then
    assert result == expected_value
    mock_service.assert_called_once_with(test_data)
```

---

## Assertions

For **small fieldsets**, assert entire dictionaries:

```python
# ✅ Good — assert whole dict
actual = some_function()
assert actual == {"field_1": "value1", "field_2": "value_2"}

# ❌ Bad — separate assertions
assert actual["field_1"] == "value1"
assert actual["field_2"] == "value2"
```

Skip whole-dict assertion when: testing one field, output is very large, or lists have non-guaranteed ordering.

---

## Mocking

### What to Mock

- Celery Tasks, HTTP calls, third-party libraries
- Django signals, injected services, side effects unrelated to main logic

### Best Practices

**Naming:** Use `mock_` prefix:

```python
mock_send_email = mocker.patch('apps.notifications.tasks.send_email')
```

**Fixture position:** `mocker` as first argument:

```python
def test__something(mocker: MockerFixture, carrier: CarrierModel) -> None:
    ...
```

**Mock location:** Mock where the object is **used**, not where it's implemented:

```python
# ✅ Good
mocker.patch('apps.accounting.use_cases.build_invoice.send_invoice_to_email')

# ❌ Bad
mocker.patch('apps.accounting.tasks.send_invoice_to_email')
```

**Celery tasks:** Mock the whole task, not just `.delay`:

```python
mock_task = mocker.patch('apps.accounting.use_cases.build_invoice.send_invoice_to_email')
mock_task.delay.assert_called_once()
```

**Shared mocks:** Use fixtures for mocks used in multiple tests:

```python
@pytest.fixture
def mock_send_email(mocker: MockerFixture) -> MagicMock:
    return mocker.patch("apps.notifications.use_cases.send_email", autospec=True)
```

---

## Feature Toggle Testing

```python
# Default disabled state (autouse)
@pytest.fixture(autouse=True)
def feature_disabled() -> Feature:
    return Feature.objects.create(name="my_feature", enabled_for_all=False)

# Explicit enabled state when needed
@pytest.fixture
def feature_enabled() -> Feature:
    feature, _ = Feature.objects.update_or_create(
        name="my_feature", defaults={"enabled_for_all": True}
    )
    return feature
```

---

## Testing Serializers/Schemas

Only test **complex validation logic**. Skip testing basic type validation that the framework handles.

```python
# ✅ Good — testing custom validation
class TestCommentSerializer:
    def test__is_valid__raises_validation_error__given_non_valid_email(self):
        serializer = CommentSerializer(data={"email": "user@gmail.com", "content": "text"})
        with pytest.raises(ValidationError):
            serializer.is_valid(raise_exception=True)
```

---

## Running Tests

<!-- CUSTOMIZE: Replace with your project's test commands -->
```bash
make test-all                                    # All tests
make test target=tests/apps/drivers/             # App tests
make test target=tests/apps/drivers/test_deactivate.py::test__execute  # Single test
```

---

## Key Principles

1. Test behavior, not implementation
2. Use descriptive test names
3. Mock external dependencies to isolate the unit
4. Keep tests focused on one specific behavior
5. Use fixtures for common setup
6. Follow Given-When-Then structure
7. Assert entire objects for small data structures
8. Only test complex validation logic for serializers
