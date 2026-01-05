# RBAC (Role-Based Access Control) System

## Overview

Role-Based Access Control (RBAC) is a security model that regulates access to system resources based on user roles within an organization. Each role aggregates a set of permissions, and users are assigned one or more roles. This design simplifies permission management, improves security, and supports auditing and compliance requirements.

---

## Key Concepts

- **User**: An individual who can access the system.
- **Role**: A collection of permissions assigned to users.
- **Permission**: An action that can be performed on a resource.
- **Role Hierarchy (optional)**: Allows roles to inherit permissions from parent roles.
- **Scope (optional)**: Contextual constraints for permissions (e.g. `SELF`, `GROUP`, `ALL`).
- **Group**: A logical collection of users.
- **Category**: A higher-level grouping of groups (e.g. departments, domains).

---

## Advantages of RBAC

- Centralized access management
- Reduced risk of human error
- Easier auditing and compliance

---

## Database Design

The RBAC system is implemented using a relational database with the following core tables:

### Main Tables

| Table | Description |
|------|------------|
| `la_user` | Stores user information (email, full name, active status, associated entity). |
| `la_role` | Stores roles (name, description). |
| `la_permission` | Stores permissions (resource, `perm_value` as bitmask). |
| `la_role_permission` | Many-to-many mapping between roles and permissions. |
| `la_role_permission_scope` | Optional scopes for role-permission assignments. |
| `la_user_role` | Many-to-many mapping between users and roles with optional expiration (`expires_at`). |
| `la_group` | User groups, optionally linked to a category. |
| `la_category` | Logical grouping of groups (e.g. departments, functional areas). |
| `la_role_hierarchy` | Optional parent-child role relationships for permission inheritance. |
---

### Entity Relationships

#### Users and Groups

- Many-to-many relationship
- Implemented via `la_user_group`
- A user can belong to multiple groups
- A group can contain multiple users

#### Users and Roles

- Many-to-many relationship via `la_user_role`
- Users may have multiple roles
- Roles may be assigned to multiple users
- Optional role expiration via `expires_at`

#### Roles and Permissions

- Many-to-many relationship via `la_role_permission`
- Permissions can be shared across roles

#### Permission Scopes

- One-to-many relationship
- Each role-permission association can define multiple scopes

#### Role Hierarchy (Optional)

- Self-referential relationship
- Implemented using `la_role_hierarchy`
- Supports permission inheritance

---

## Local Database Configuration

### Configuration File

**`config.local.kube.yaml`**

```yaml
DATABASE:
  USERNAME: "root"
  PASSWORD: "admin"
  NAME: "leonard_test"
  HOST: "localhost"
  PORT: "3306"
```

### Environment Setup (`env.py`)

```python
from common.db.database import Base
from common.utils.config_utils import AppSettings
from common.utils.vault import AzureVault

config = AppSettings()

try:
    KEY_VAULT_URI = config.get_setting("KEYVAULT.URI")
    KEY_VAULT_NAME = config.get_setting("KEYVAULT.NAME")
    vault = AzureVault(KEY_VAULT_URI, KEY_VAULT_NAME)

    DB_USERNAME = vault.get_secret(config.get_setting("DATABASE.USERNAME_SECRET_KEY"))
    DB_PASSWORD = vault.get_secret(config.get_setting("DATABASE.PASSWORD_SECRET_KEY"))
    DB_NAME = vault.get_secret(config.get_setting("DATABASE.NAME_SECRET_KEY"))
    DB_HOST = config.get_setting("DATABASE.HOST")
    DB_PORT = config.get_setting("DATABASE.PORT")
except Exception:
    # Local development fallback
    DB_USERNAME = config.get_setting("DATABASE.USERNAME")
    DB_PASSWORD = config.get_setting("DATABASE.PASSWORD")
    DB_NAME = config.get_setting("DATABASE.NAME")
    DB_HOST = config.get_setting("DATABASE.HOST")
    DB_PORT = config.get_setting("DATABASE.PORT")
```

### SQLAlchemy Engine (`database.py`)

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

DATABASE_URL = f"mysql+pymysql://{DB_USERNAME}:{DB_PASSWORD}@{DB_HOST}:{DB_PORT}/{DB_NAME}"

engine = create_engine(DATABASE_URL, echo=True, future=True)
SessionLocal = sessionmaker(bind=engine, autocommit=False, autoflush=False)
```

---

## Alembic Migrations

### Create Tables

```bash
alembic revision --autogenerate -m "create RBAC tables"
alembic upgrade head
```

**Tables created:**

- `la_user`
- `la_group`
- `la_role`
- `la_permission`
- `la_user_role`
- `la_role_permission`
- `la_role_permission_scope`

All tables include:

- Primary keys
- Foreign keys
- Cascade delete rules

---

### Seed Test Data

```sql
INSERT INTO la_category (name, description)
VALUES ('HR', 'Human Resources related groups');

INSERT INTO la_group (name, category_id)
VALUES ('HR_MANAGERS', 1);

INSERT INTO la_user (email, full_name, is_active)
VALUES ('louis.bertrand@quanteam.fr', 'Louis BERTRAND', true);

INSERT INTO la_role (name)
VALUES ('Consultant Senior');
```

---

## Usage Examples

```python
from common.dao.user_dao import UserDAO

user_dao = UserDAO()

# Active users
active_users = user_dao.get_active_users()
print(f"Active users count: {len(active_users)}")

# All users with eager loading
all_users = user_dao.get_all(load_tree=user_dao.DEFAULT_LOAD_TREE)

# Users in a specific group
test_group_users = user_dao.get_users_in_group_by_name("HR_MANAGERS")

# Roles with expiry for a specific user
roles_expiry = user_dao.get_roles_with_expiry(user_id=10)

# Users belonging to a specific entity
entity_users = user_dao.get_all_for_entity(entity_id=4)
```

---

## Notes

- Use `load_tree` for flexible eager-loading.
- Use `main_filters: list` or `related_filters: dict` in `get_all` for database-side filtering.
- Session management is handled via the `@db_operation` decorator.
- `.unique()` is applied internally to avoid duplicates when joining collections.

---

## Diagrams

- **ER Diagram**: `Untitled diagram-2025-12-04-104041.png`
- **UML Diagram (Mermaid)**: `Mermaid Chart - Create complex, visual diagrams with text.-2025-12-01-151315.png`

These diagrams provide a visual overview of the RBAC entities and their relationships.

