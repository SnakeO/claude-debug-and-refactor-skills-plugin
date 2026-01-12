---
name: debug:django
description: Debug Django web applications with systematic diagnostic approaches. This skill covers troubleshooting Django-specific errors including TemplateDoesNotExist, ImproperlyConfigured, IntegrityError, migration conflicts, CSRF failures, N+1 query problems, and circular imports. Includes Django Debug Toolbar setup, ORM query logging, pdb/ipdb usage, shell_plus debugging, and comprehensive logging configuration. Provides four-phase methodology for root cause analysis and regression prevention.
---

# Django Debugging Guide

## Overview

Django debugging requires understanding the framework's layered architecture: settings, URL routing, views, templates, models, and middleware. Effective debugging follows a systematic approach - reproducing the bug, isolating the problem, gathering evidence, and implementing verified fixes.

Key principles:
- **Reproduce first**: Repeat the exact steps that cause the issue
- **Isolate systematically**: Identify the exact module, function, or code path
- **Gather evidence**: Collect error messages, stack traces, logs, and database state
- **Test your fix**: Verify the solution with automated tests

## Common Error Patterns

### TemplateDoesNotExist
**Cause**: Django cannot find the specified template file.

**Investigation steps**:
1. Check template exists in the correct directory
2. Verify `TEMPLATES['DIRS']` setting includes your template directories
3. Check for typos in template path
4. Ensure app is in `INSTALLED_APPS` if using app-specific templates

```python
# Check template configuration
python manage.py shell -c "from django.conf import settings; print(settings.TEMPLATES)"

# Verify template directories exist
python manage.py shell -c "import os; from django.conf import settings; print([d for d in settings.TEMPLATES[0]['DIRS'] if os.path.exists(d)])"
```

### ImproperlyConfigured
**Cause**: Django configuration error in settings.py or environment.

**Common causes**:
- Missing or incorrect `DJANGO_SETTINGS_MODULE`
- Invalid database configuration
- Missing required settings
- Incorrect app configuration

```bash
# Check settings module
echo $DJANGO_SETTINGS_MODULE

# Validate configuration
python manage.py check

# Compare with defaults
python manage.py diffsettings
```

### IntegrityError / Database Errors
**Cause**: Database constraint violations or missing migrations.

**OperationalError (missing column/table)**:
```bash
# Check migration status
python manage.py showmigrations

# Create missing migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Check SQL that would be executed
python manage.py sqlmigrate <app_name> <migration_number>
```

**IntegrityError (constraint violation)**:
```python
# Check for duplicates before insert
from django.db import IntegrityError

try:
    obj.save()
except IntegrityError as e:
    # Log the error, check constraint name
    print(f"Constraint violation: {e}")
```

### N+1 Query Problems
**Cause**: Inefficient database queries in loops.

**Detection**:
```python
# In settings.py (development only)
LOGGING = {
    'version': 1,
    'handlers': {'console': {'class': 'logging.StreamHandler'}},
    'loggers': {
        'django.db.backends': {
            'level': 'DEBUG',
            'handlers': ['console'],
        }
    }
}
```

**Solution - use select_related/prefetch_related**:
```python
# Bad: N+1 queries
for order in Order.objects.all():
    print(order.customer.name)  # Extra query per order

# Good: Single query with join
for order in Order.objects.select_related('customer'):
    print(order.customer.name)

# For many-to-many or reverse foreign keys
orders = Order.objects.prefetch_related('items')
```

### Migration Conflicts
**Cause**: Multiple developers creating migrations on same app.

```bash
# View migration graph
python manage.py showmigrations --plan

# Merge conflicting migrations
python manage.py makemigrations --merge

# Reset migrations (development only - DANGEROUS)
python manage.py migrate <app_name> zero
python manage.py makemigrations <app_name>
python manage.py migrate <app_name>
```

### CSRF Verification Failures
**Cause**: Missing CSRF token or misconfigured trusted origins.

**Checklist**:
1. Include `{% csrf_token %}` in forms
2. Check `CSRF_TRUSTED_ORIGINS` for HTTPS sites
3. Verify middleware includes `CsrfViewMiddleware`
4. For AJAX, include CSRF token in headers

```python
# settings.py - for cross-origin requests
CSRF_TRUSTED_ORIGINS = [
    'https://your-domain.com',
]
```

### Import Errors / Circular Imports
**Cause**: Circular dependencies between modules.

**Detection**:
```bash
# Check for import errors
python -c "import your_app"

# Verbose import tracing
python -v -c "import your_app" 2>&1 | grep "import"
```

**Solutions**:
- Move imports inside functions (lazy import)
- Restructure modules to break cycles
- Use string references for model relationships

```python
# Avoid circular import with lazy import
def my_function():
    from other_module import SomeClass  # Import inside function
    return SomeClass()

# Use string reference in ForeignKey
class Order(models.Model):
    customer = models.ForeignKey('customers.Customer', on_delete=models.CASCADE)
```

### DoesNotExist
**Cause**: Querying for an object that does not exist in the database.

```python
# Bad: Raises DoesNotExist
user = User.objects.get(id=999)

# Good: Handle missing objects
from django.shortcuts import get_object_or_404
user = get_object_or_404(User, id=999)

# Or use get_or_create
user, created = User.objects.get_or_create(
    username='john',
    defaults={'email': 'john@example.com'}
)

# Or wrap in try-except
try:
    user = User.objects.get(id=999)
except User.DoesNotExist:
    user = None
```

### DisallowedHost
**Cause**: Request host not in ALLOWED_HOSTS setting.

```python
# settings.py
ALLOWED_HOSTS = [
    'localhost',
    '127.0.0.1',
    'your-domain.com',
    '.your-domain.com',  # Wildcard for subdomains
]
```

### NoReverseMatch
**Cause**: URL reverse lookup failed - URL name not found or wrong arguments.

```bash
# List all URL patterns
python manage.py show_urls  # requires django-extensions

# Or manually inspect
python manage.py shell -c "from django.urls import get_resolver; print([p.name for p in get_resolver().url_patterns])"
```

```python
# Check URL name matches exactly
from django.urls import reverse
url = reverse('app_name:view_name', args=[object_id])
url = reverse('app_name:view_name', kwargs={'pk': object_id})
```

## Debugging Tools

### Django Debug Toolbar
The most powerful visual debugging tool for Django development.

**Installation**:
```bash
pip install django-debug-toolbar
```

**Configuration**:
```python
# settings.py (development only)
INSTALLED_APPS = [
    # ...
    'debug_toolbar',
]

MIDDLEWARE = [
    'debug_toolbar.middleware.DebugToolbarMiddleware',
    # ... other middleware
]

INTERNAL_IPS = ['127.0.0.1']

# For Docker
DEBUG_TOOLBAR_CONFIG = {
    'SHOW_TOOLBAR_CALLBACK': 'debug_toolbar.middleware.show_toolbar_with_docker',
}
```

```python
# urls.py
from debug_toolbar.toolbar import debug_toolbar_urls

urlpatterns = [
    # ... your URLs
] + debug_toolbar_urls()
```

**Key panels**:
- **SQL**: All database queries with timing and EXPLAIN
- **Templates**: Rendered templates and context variables
- **Cache**: Cache hits/misses
- **Request**: Headers, cookies, session data
- **Signals**: Django signals fired

### Python Debugger (pdb/ipdb)

```python
# Insert breakpoint in code
breakpoint()  # Python 3.7+ (uses PYTHONBREAKPOINT env var)

# Or explicitly
import pdb; pdb.set_trace()

# For better experience
import ipdb; ipdb.set_trace()
```

**Common pdb commands**:
- `n` (next): Execute next line
- `s` (step): Step into function
- `c` (continue): Continue execution
- `p variable`: Print variable value
- `pp variable`: Pretty print
- `l` (list): Show current code
- `w` (where): Show stack trace
- `q` (quit): Exit debugger

### django-extensions shell_plus

```bash
pip install django-extensions

# Add to INSTALLED_APPS
INSTALLED_APPS = ['django_extensions', ...]

# Use enhanced shell with auto-imports
python manage.py shell_plus

# With IPython
python manage.py shell_plus --ipython

# Print SQL queries
python manage.py shell_plus --print-sql
```

### Logging Configuration

```python
# settings.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
        'file': {
            'class': 'logging.FileHandler',
            'filename': 'debug.log',
            'formatter': 'verbose',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['console'],
            'level': 'INFO',
        },
        'django.db.backends': {
            'handlers': ['console'],
            'level': 'DEBUG',  # Shows all SQL queries
        },
        'myapp': {
            'handlers': ['console', 'file'],
            'level': 'DEBUG',
        },
    },
}
```

**Usage in code**:
```python
import logging
logger = logging.getLogger(__name__)

logger.debug('Detailed information')
logger.info('General information')
logger.warning('Warning message')
logger.error('Error occurred', exc_info=True)  # Include traceback
logger.exception('Exception occurred')  # Auto-includes traceback
```

### SQL Query Logging

```python
# In Django shell
from django.db import connection
from django.db import reset_queries

reset_queries()
# ... execute your queries ...
print(connection.queries)

# Or use QuerySet.explain()
queryset = User.objects.filter(is_active=True)
print(queryset.explain())
```

## The Four Phases (Django-Specific)

### Phase 1: Root Cause Investigation

**Gather evidence systematically**:

```bash
# 1. Check Django configuration
python manage.py check
python manage.py check --deploy  # Production security checks

# 2. Review migration state
python manage.py showmigrations
python manage.py showmigrations --plan  # Execution order

# 3. Inspect settings
python manage.py diffsettings  # Compare with Django defaults

# 4. Check logs
tail -f debug.log
docker compose logs -f api  # If using Docker
```

**Examine the error traceback**:
- Note the exception type (determines error category)
- Find the last line of YOUR code (not Django internals)
- Check variable values in the frame

**Database state inspection**:
```python
# In shell
python manage.py shell

from myapp.models import MyModel
MyModel.objects.count()
MyModel.objects.filter(status='error').values()
```

### Phase 2: Pattern Analysis

**Compare with working patterns**:

1. **Check similar working code**: Find views/functions that work and compare
2. **Review Django documentation**: Verify correct usage of APIs
3. **Check third-party package versions**: Ensure compatibility

```bash
# Check installed versions
pip freeze | grep -i django
pip show django-rest-framework
```

**Common pattern mismatches**:
- Missing `return` in view functions
- Wrong queryset method order
- Incorrect URL pattern syntax
- Missing model field attributes

### Phase 3: Hypothesis and Testing

**Write a failing test first**:

```python
# tests/test_debug.py
import pytest
from django.test import TestCase, Client

@pytest.mark.django_db
class TestBugFix(TestCase):
    def test_reproduces_bug(self):
        """This test should fail until bug is fixed."""
        client = Client()
        response = client.get('/problematic-url/')
        self.assertEqual(response.status_code, 200)
        # Add assertions that currently fail

    def test_edge_case(self):
        """Test the specific input that causes the bug."""
        from myapp.services import problematic_function
        result = problematic_function(edge_case_input)
        self.assertEqual(result, expected_output)
```

**Run tests in isolation**:

```bash
# Run single test
pytest tests/test_debug.py::TestBugFix::test_reproduces_bug -v

# Run with verbose SQL output
pytest tests/test_debug.py -v --capture=no

# Run with debugger on failure
pytest tests/test_debug.py --pdb

# Run with coverage
pytest tests/test_debug.py --cov=myapp --cov-report=term-missing
```

### Phase 4: Implementation

**Implement the fix**:

1. Make minimal changes to fix the issue
2. Ensure existing tests still pass
3. Add new tests for the specific bug
4. Document the fix in commit message

**Verify the fix**:

```bash
# Run all tests
pytest -v

# Run specific test file
pytest tests/test_affected_module.py -v

# Check for regressions
pytest tests/ -v --tb=short

# Verify in development server
python manage.py runserver
```

**Post-fix checklist**:
- [ ] All tests pass
- [ ] No new warnings introduced
- [ ] `python manage.py check` passes
- [ ] Related functionality manually tested
- [ ] Code reviewed

## Quick Reference Commands

### Django Management Commands

```bash
# System checks
python manage.py check                    # Run system checks
python manage.py check --deploy           # Production deployment checks
python manage.py check --tag security     # Security-specific checks

# Migrations
python manage.py showmigrations           # List all migrations and status
python manage.py showmigrations --plan    # Show execution order
python manage.py sqlmigrate app 0001      # Show SQL for migration
python manage.py makemigrations --dry-run # Preview migration creation
python manage.py migrate --plan           # Show migration plan

# Settings
python manage.py diffsettings             # Show non-default settings
python manage.py diffsettings --all       # Show all settings

# Database
python manage.py dbshell                  # Database shell
python manage.py inspectdb                # Generate models from database

# Shell
python manage.py shell                    # Django shell
python manage.py shell_plus               # Enhanced shell (django-extensions)
python manage.py shell_plus --print-sql   # Shell with SQL logging

# URLs
python manage.py show_urls                # List all URLs (django-extensions)

# Cleanup
python manage.py clearsessions            # Clear expired sessions
python manage.py flush                    # Clear database (DANGEROUS)
```

### Quick Debugging Snippets

```python
# Print all SQL queries in view
from django.db import connection
def my_view(request):
    # ... view logic ...
    for query in connection.queries:
        print(query['sql'])

# Check if request is AJAX
request.headers.get('X-Requested-With') == 'XMLHttpRequest'

# Inspect request data
print(request.GET.dict())
print(request.POST.dict())
print(request.body)
print(dict(request.headers))

# Debug template context
# In template:
{% debug %}

# Or in view:
from django.template import engines
engine = engines['django']
template = engine.from_string("{{ debug }}")
```

### Environment Debugging

```bash
# Check Python environment
which python
python --version
pip list

# Check Django version
python -c "import django; print(django.VERSION)"

# Check database connection
python manage.py dbshell -c "SELECT 1;"

# Check Redis connection (if using)
redis-cli ping

# Check Celery workers
celery -A config inspect active
celery -A config inspect stats
```

## Additional Resources

- [Django Debug Toolbar Documentation](https://django-debug-toolbar.readthedocs.io/)
- [Django Logging Documentation](https://docs.djangoproject.com/en/5.1/topics/logging/)
- [Django Exceptions Reference](https://docs.djangoproject.com/en/5.2/ref/exceptions/)
- [Common Django Errors and Solutions](https://medium.com/@djangowiki/common-django-errors-and-how-to-fix-them-a-practical-guide-16b80fd007f6)
- [Django Error Handling Patterns](https://betterstack.com/community/guides/scaling-python/error-handling-django/)
- [Debugging Tips and Techniques by Matt Layman](https://www.mattlayman.com/understand-django/debugging-tips-techniques/)
