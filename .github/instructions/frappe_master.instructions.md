---
description: "Frappe Press project master context. Use when: working on any Frappe Press code, managing multiple Frappe applications, setting up documentation, or creating infrastructure code. Provides project architecture, workflows, and when to use specialized instructions (base.instructions.md for Python)."
applyTo: |
  **/*.py
  **/*.js
  **/*.md
  **/*.json
  **/*.html
  .github/**
---

# Frappe Press Master Instructions & Project Context

Frappe Press is an all-in-one platform for managing multiple Frappe applications. This file provides project architecture, key concepts, and development workflows.

## Project Overview

**What is Frappe Press?**
- A management platform that simplifies deploying, scaling, and maintaining multiple Frappe applications
- Built on Frappe framework (ERP platform based on Python/JavaScript)
- Acts as a "control plane" for managing Frappe instances, similar to Frappe Cloud
- Handles multi-tenancy, resource allocation, monitoring, and lifecycle management of Frappe apps

**Key Components:**
- **Bench Management**: Create, configure, and update Frappe benches (development environments)
- **Site Management**: Deploy and manage Frappe sites (instances) across multiple benches
- **Resource Management**: Allocate and monitor resources (database, cache, storage)
- **Authentication & Authorization**: Multi-tenant security and role-based access
- **Monitoring & Logging**: Track application health and system metrics
- **Migration & Updates**: Handle version upgrades and data migrations across apps

## When to Use base.instructions.md

The **base.instructions.md** file applies to **all Python files** (`**/*.py`). Load it when:
- Writing or reviewing any Python code in the project
- Working on Frappe doctypes, methods, or RPC endpoints
- Creating migrations, hooks, or backend logic
- Testing Python functionality
- The agent should enforce PEP 8, type hints (PEP 484), Frappe patterns, and testing standards

**⚠️ Important**: Python code MUST follow base.instructions.md standards in addition to these master guidelines.

## Frappe-Specific Task Workflows

### 1. Creating a New Doctype

```
1. Navigate to: `app/doctype_name_folder/`
2. Create three files:
   - `doctype_name.py` → Master doctype class
   - `doctype_name.json` → Doctype metadata (fields, permissions)
   - `doctype_name_list.js` → Optional list view customization
3. Implement lifecycle methods:
   - `validate()` → Run before save
   - `before_insert()` → Run before first save
   - `on_update()` → Run after save
   - `before_delete()` → Run before deletion
4. Export metadata: Run `bench export-fixtures`
5. Add doctype to fixtures in hooks.py
```

**Pattern:**
```python
from frappe import _
from frappe.model.document import Document

class MyDoctype(Document):
    """DocString explaining the doctype's purpose."""
    
    def validate(self):
        """Validate document before save."""
        ...
```

### 2. Creating an RPC Method (API Endpoint)

```
1. Create method in doctype class or in `app/methods.py`
2. Decorate with @frappe.whitelist() for public access
3. Use auto_commit=True for side effects that modify state
4. Always validate user permissions
5. Return JSON-serializable data
6. Handle errors with frappe.throw() for user errors
```

**Pattern:**
```python
@frappe.whitelist()
def my_rpc_method(name: str, value: str) -> Dict[str, Any]:
    """Description of what the RPC method does.
    
    Args:
        name: The document name
        value: The value to set
        
    Returns:
        Dictionary with result data
        
    Raises:
        frappe.ValidationError: If validation fails
    """
    if not frappe.has_permission("DocType", "read", name):
        frappe.throw("You don't have permission", frappe.PermissionError)
    
    doc = frappe.get_doc("DocType", name)
    doc.field_name = value
    doc.save()
    return {"status": "success"}
```

### 3. Database Migrations (Patches)

```
1. Create file: `app/patches/v{version}__{short_description}.py`
2. Implement execute() function that modifies database/documents
3. Link to issue tracker in docstring
4. Use frappe.db for raw SQL or frappe.get_doc() for ORM
5. Patches run once per environment on next bench migrate
```

**Pattern:**
```python
"""
Patch: My feature description
Related: GitHub issue #123
"""

def execute():
    """Execute migration."""
    frappe.db.sql("ALTER TABLE `tab{DocType}` ADD COLUMN new_field VARCHAR(255)")
    frappe.db.commit()
```

### 4. Frappe Hooks Configuration

```
1. File: `app/hooks.py`
2. Define app metadata (version, title, icon)
3. Register doctypes, methods, permissions
4. Hook into Frappe lifecycle events (before_insert, after_migrate, etc.)
5. Configure fixtures (initial data)
6. Setup scheduled jobs (background tasks)
```

**Common Hooks Patterns:**
```python
app_name = "my_app"
app_title = "My App Title"
app_version = "1.0.0"

# Doctypes to display immediately
default_sidebar_items = ["My DocType"]

# Fixtures (data that syncs on migrate)
fixtures = [{"doctype": "Custom Role", "filters": [["name", "in", ["My Role"]]]}]

# RPC methods
methods = {
    "my_app.my_module.my_rpc_method": {"timeout": 60}
}

# Scheduled jobs (cron)
scheduler_events = {
    "hourly": ["my_app.tasks.hourly_task"],
    "daily": ["my_app.tasks.daily_task"]
}

# Lifecycle hooks
doc_events = {
    "DocType": {
        "before_insert": ["my_app.handlers.validate_doctype"],
        "on_update": ["my_app.handlers.sync_external_system"]
    }
}
```

## Common Patterns & Best Practices

### Error Handling
- Use `frappe.throw()` for user-facing errors (will show in UI)
- Use `raise Exception()` for system/developer errors (will log)
- Always specify error class: `frappe.throw("Message", frappe.ValidationError)`

### Permissions
- Always check `frappe.has_permission()` before allowing operations
- Use `frappe.only_for()` to restrict methods to specific roles
- Validate both document-level and field-level permissions

### Database Best Practices
- Use `frappe.get_all()` for list queries (efficient)
- Use `frappe.get_doc()` for single document access (loads all fields)
- Use `frappe.db.sql()` for complex queries (escape with %(key)s binding)
- Always call `frappe.db.commit()` after modifications if not using auto_commit

### Configuration & Settings
- Store configuration in DocType with Settings doctype
- Use `frappe.get_config_value("app_name", "key")` to access app config
- Never hardcode values—use settings or environment variables

## Project Structure

```
frappe-press/
├── .github/
│   ├── instructions/           # Agent instructions
│   │   ├── base.instructions.md      # Python standards (applies to **/*.py)
│   │   └── frappe_master.instructions.md  # This file (applies to all files)
│   └── workflows/              # GitHub Actions CI/CD
├── app/                        # Main application
│   ├── doctype/                # Frappe doctypes
│   ├── methods/                # API methods
│   ├── hooks.py                # App configuration
│   ├── config.py               # Menu configuration
│   └── patches/                # Database migrations
├── tests/                      # Test suite (pytest)
├── local/                      # Local development guides
├── README.md                   # Project overview
└── pyproject.toml              # Python dependencies & config
```

## Frappe Press-Specific Gotchas

### 1. Multi-Tenancy Context
- Always consider that operations may run in multi-tenant environments
- Site-specific data must be isolated per site (use frappe.defaults)
- Global operations must be carefully coordinated

### 2. Background Jobs
- Long-running tasks should use Frappe's job queue (frappe.enqueue())
- Don't block RPC responses with heavy operations
- Set appropriate timeouts and retry policies

### 3. Version Compatibility
- Frappe Press may manage multiple Frappe versions
- Test patches and features with target Frappe versions
- Use version guards when needed: `frappe.get_attr("frappe.version", "")`

### 4. Infrastructure Code
- Non-Python code (Terraform, Docker, JavaScript) may have different standards
- Always follow language-specific conventions
- Document infrastructure changes thoroughly

## Links & References

- [Frappe Documentation](https://frappeframework.com/)
- [Frappe Press Official](https://frappe.io/press)
- [Local Frappe Cloud Setup Guide](https://docs.frappe.io/cloud/local-fc-setup)
- [Frappe GitHub Repository](https://github.com/frappe/frappe)

---

**Next Steps**: 
- For Python code details → Use `base.instructions.md` as primary reference
- For code structure → Follow project structure above
- For domain questions → Use this master context file as reference