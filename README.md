# RBAC (Role-Based Access Control) System

## Overview

Role-Based Access Control (RBAC) is a security model that regulates access to system resources based on user roles within an organization. Each role aggregates a set of permissions, and users are assigned one or more roles. This design simplifies permission management, improves security, and supports auditing and compliance requirements.

---

## Key Concepts

* **User**: An individual who can access the system.
* **Role**: A collection of permissions assigned to users.
* **Permission**: An action that can be performed on a resource.
* **Role Hierarchy (optional)**: Allows roles to inherit permissions from parent roles.
* **Scope (optional)**: Contextual constraints for permissions (e.g. `SELF`, `GROUP`, `ALL`).
* **Group**: A logical collection of users.
* **Category**: A higher-level grouping of groups (e.g. departments, domains).

---

## Advantages of RBAC

* Centralized access management
* Reduced risk of human error
* Easier auditing and compliance

---

## Database Design

The RBAC system is implemented using a relational database with the following core tables:

### Main Tables

| Table                      | Description                                                                           |
| -------------------------- | ------------------------------------------------------------------------------------- |
| `la_user`                  | Stores user information (email, full name, active status, associated entity).         |
| `la_role`                  | Stores roles (name, description).                                                     |
| `la_permission`            | Stores permissions (resource, `perm_value` as bitmask).                               |
| `la_role_permission`       | Many-to-many mapping between roles and permissions.                                   |
| `la_role_permission_scope` | Optional scopes for role-permission assignments.                                      |
| `la_user_role`             | Many-to-many mapping between users and roles with optional expiration (`expires_at`). |
| `la_group`                 | User groups, optionally linked to a category.                                         |
| `la_category`              | Logical grouping of groups (e.g. departments, functional areas).                      |
| `la_role_hierarchy`        | Optional parent-child role relationships for permission inheritance.                  |

---

### Entity Relationships

#### Users and Groups

* Many-to-many relationship
* Implemented via `la_user_group`
* A user can belong to multiple groups
* A group can contain multiple users

#### Users and Roles

* Many-to-many relationship via `la_user_role`
* Users may have multiple roles
* Roles may be assigned to multiple users
* Optional role expiration via `expires_at`

#### Roles and Permissions

* Many-to-many relationship via `la_role_permission`
* Permissions can be shared across roles

#### Permission Scopes

* One-to-many relationship
* Each role-permission association can define multiple scopes

#### Role Hierarchy (Optional)

* Self-referential relationship
* Implemented using `la_role_hierarchy`
* Supports permission inheritance

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
    KEY
```
