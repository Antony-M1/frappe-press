---
description: Frappe Press project conventions. Use when working with Python code in this Frappe-based platform project. Enforces PEP 8, type hints (PEP 484), Zen principles (PEP 20), Frappe patterns, and testing standards.
applyTo: '**/*.py'
---

# Frappe Press Python Standards & Conventions

This project is a Frappe-based platform for managing multiple Frappe applications. All Python code must follow strict PEP standards and Frappe conventions.

## Python Code Standards

### Style Guide (PEP 8)
- **Line length**: Maximum 88 characters (black formatter compatible)
- **Indentation**: 4 spaces, never tabs
- **Naming conventions**:
  - Classes: `PascalCase` (e.g., `UserProfile`)
  - Functions/methods: `snake_case` (e.g., `get_user_profile()`)
  - Constants: `UPPER_SNAKE_CASE` (e.g., `MAX_RETRY_ATTEMPTS`)
  - Private methods/attributes: Prefix with `_` (e.g., `_internal_helper()`)
- **Imports**: 
  - Group in order: standard library, third-party, local imports
  - Use `from X import Y` over `import X` for clarity
  - Avoid wildcard imports (`from X import *`)
- **Whitespace**:
  - Two blank lines between top-level definitions
  - One blank line between method definitions
  - No trailing whitespace

### Type Hints (PEP 484)
- **Required for**:
  - All function/method parameters and return types
  - Class attributes (in `__init__` or class docstring)
  - Public API functions
- **Format**:
  ```python
  def process_document(doc: Dict[str, Any], validate: bool = True) -> bool:
      """Process a document with optional validation."""
      pass
  ```
- **Use**: `Optional[T]` for nullable types, `Union[T, U]` for multiple types, `List`, `Dict`, `Tuple` from `typing`

### Zen of Python (PEP 20)
Apply to all design decisions:
- **Explicit is better than implicit**: Clear variable names, explicit error handling
- **Simple is better than complex**: Prefer straightforward logic over clever shortcuts
- **Readability counts**: Write code for humans first
- **Errors should never pass silently**: Always handle exceptions explicitly
- **Special cases aren't special enough to break the rules**: Follow conventions consistently

## Frappe-Specific Conventions

### Docstring Format
Use the Frappe/Google style for all docstrings:
```python
def get_user_settings(user_id: str) -> Dict[str, Any]:
    """Retrieve user settings from the database.
    
    Args:
        user_id: The unique identifier for the user
        
    Returns:
        Dictionary containing user preferences and settings
        
    Raises:
        ValueError: If user_id is empty or invalid
        frappe.DoesNotExistError: If user does not exist
    """
    pass
```

### Document Handling
- Use `frappe.get_doc()` to fetch documents
- Use `doc.save()` for persistence (not raw SQL)
- Always check permissions with `frappe.has_permission()`
- Validate documents before saving: call `doc.validate()` explicitly or set auto-validate
- Use `frappe.throw()` for user-facing errors, `raise` for system errors

### Method Naming
- Frappe RPC methods should be prefixed with `@frappe.whitelist()` and use `auto_commit=True` when needed
- Use clear names: `get_`, `create_`, `update_`, `delete_`, `validate_` for their respective actions
- Private helper methods: prefix with `_internal_` or `_`

### Error Handling
```python
try:
    doc = frappe.get_doc("Document", name)
except frappe.DoesNotExistError:
    frappe.throw(f"Document {name} not found", frappe.DoesNotExistError)
```

## Testing Standards

### Framework: pytest
- Test files: `test_*.py` or `*_test.py` in `tests/` directory
- Test function naming: `test_<function>_<scenario>` (e.g., `test_get_user_settings_valid_user`)
- Use fixtures for common setup/teardown

### Frappe Testing Utilities
- Use `frappe.core.doctype.*` utilities for document testing
- Clean up test data in teardown (use fixtures or `frappe.db.rollback()`)
- Mock external API calls with `unittest.mock` or `responses` library
- Test both happy path and error cases

### Doctests
- Include examples in docstrings when helpful
- Format as Python REPL sessions:
  ```python
  """
  Example:
      >>> get_user_settings("user123")
      {'theme': 'dark', 'language': 'en'}
  """
  ```

## Code Organization

### Module Structure
- **Logical grouping**: Related functions in same module
- **Single responsibility**: Each module has one clear purpose
- **Utils separation**: Avoid dumping all helpers in `utils.py`; create focused modules (`validators.py`, `formatters.py`, etc.)

### Imports
```python
# Standard library
import json
from typing import Any, Dict, List, Optional

# Third-party
import requests
import frappe
from frappe import _

# Local imports
from .helpers import format_date
from .validators import validate_email
```

### File Organization
- Maximum 500 lines per file (split into logical modules if larger)
- Related classes in same file only if tightly coupled
- Separate config from logic

## Common Patterns

### Document Validation
```python
class Document(frappe.Document):
    def validate(self):
        """Validate document data."""
        self._validate_required_fields()
        self._validate_email_format()
    
    def _validate_required_fields(self) -> None:
        """Check all required fields are present."""
        required = ["email", "name"]
        for field in required:
            if not self.get(field):
                frappe.throw(f"Field '{field}' is mandatory")
    
    def _validate_email_format(self) -> None:
        """Validate email format if provided."""
        if self.email and not is_valid_email(self.email):
            frappe.throw(f"Invalid email: {self.email}")
```

### Permission Checks
```python
@frappe.whitelist()
def get_sensitive_data(doc_name: str) -> Dict[str, Any]:
    """Retrieve sensitive document data."""
    if not frappe.has_permission("Document", "read", doc_name):
        frappe.throw("Insufficient permissions", frappe.PermissionError)
    
    doc = frappe.get_doc("Document", doc_name)
    return doc.as_dict()
```

## Code Review Checklist

When submitting changes, verify:
- ✅ All functions have type hints (parameters and return type)
- ✅ Docstrings present for public functions and classes
- ✅ PEP 8 compliance (use `black` formatter if available)
- ✅ No wildcard imports
- ✅ Proper error handling with `frappe.throw()` for user errors
- ✅ Tests added for new functionality
- ✅ No hardcoded values (use settings/config)
- ✅ Database queries use ORM (`frappe.get_doc`, `frappe.get_list`)
- ✅ Permissions checked on sensitive operations
- ✅ Follows Frappe patterns for RPC methods and validators

## Tools & Automation

Recommended tools for this project:
- **Formatter**: `black` (run: `black --line-length=88 .`)
- **Linter**: `pylint` with `.pylintrc` configuration
- **Type checker**: `mypy` for static type validation
- **Test runner**: `pytest` with coverage reporting

## References

- [PEP 8 Style Guide](https://www.python.org/dev/peps/pep-0008/)
- [PEP 20 Zen of Python](https://www.python.org/dev/peps/pep-0020/)
- [PEP 484 Type Hints](https://www.python.org/dev/peps/pep-0484/)
- [Frappe Framework Documentation](https://frappeframework.com/)
- [Frappe Bench](https://frappe.io/bench)