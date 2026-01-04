# Tech Stack

## Backend Technologies

| Technology | Purpose |
|----------|---------|
| Django | Core backend framework |
| Django REST Framework | REST API development |
| PostgreSQL | Relational database |
| JWT | Token-based authentication |
| MkDocs | Project documentation |

---

## Key Integrations

### JWT Authentication with Token Refresh
- Stateless authentication
- Access and refresh token lifecycle
- Secure API access

---

### Role-Based Access Control (RBAC)
- Users and Admins share the same endpoints
- Permissions control behavior
- Querysets restrict data access

---

### Soft Delete Pattern
- Records are not physically deleted
- `deleted_at` timestamp marks inactive data
- Prevents accidental data loss
