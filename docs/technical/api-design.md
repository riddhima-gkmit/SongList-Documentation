# API Design

The **SongList API** follows **RESTful architecture principles**, providing clean, resource-based endpoints for authentication, user management, song moderation, and playlist management.

All responses are returned in **JSON format**, and all protected endpoints require **JWT-based authentication**.

---

## Base Configuration

* **Base URL:** `http://localhost:8000/api/v1`
* **Authentication:** JWT (Access & Refresh Tokens)
* **Core Rule:**

  * **Same resource → same endpoint**
  * **Different access → permissions + queryset filtering**

No role-specific or admin-specific URLs are used.

---

## Role Definitions (Authentication APIs)

### All Users
Anyone accessing the system, including unauthenticated users.
Used only for public authentication endpoints.

### Authenticated User
Any logged-in user (User or Admin) authenticated via a valid JWT token.
Used where only login is required, and behavior is identical for all roles.

### User / Admin
Any logged-in user.
Used where both roles can access the endpoint, but behavior or visibility differs (e.g., queryset filtering).

### Owner / Admin
The resource owner or an Admin.
Used where object-level ownership enforcement is required.

## Authentication Endpoints

| Role               | Method | Endpoint                       | Description                                     |
| ------------------ | ------ | ------------------------------ | ----------------------------------------------- |
| All Users          | POST   | `/api/v1/auth/register/`        | Register a new user. Role is `USER` by default. |
| All Users          | POST   | `/api/v1/auth/login/`           | Authenticate and receive JWT tokens.            |
| All Users          | POST   | `/api/v1/token/refresh/`        | Refresh JWT access token.                       |
| Authenticated User | POST   | `/api/v1/auth/logout/`          | Logout and invalidate refresh token.            |

---

## User Endpoints (Self-Service)

Users manage their own profile through a **single self-service endpoint**.

| Role               | Method | Endpoint           | Description                            |
| ------------------ | ------ | ------------------ | -------------------------------------- |
| Authenticated User | GET    | `/api/v1/users/me/` | Retrieve current user profile details. |
| Authenticated User | POST | `/api/v1/users/me/change-password/` | Change password for logged-in user. |
| Authenticated User | PATCH    | `/api/v1/users/me/` | Update own profile information.        |
| User | DELETE | `/api/v1/users/me/` | Delete own user account (soft delete). |

### Notes

- No `/users/{id}` endpoint is exposed for regular users.
- Admin-level access is enforced through permissions, not URL structure.
- This design prevents unauthorized access to other users’ data.
---

## User Management Endpoints (Admin Access)

These endpoints allow **Admin** to manage all users in the system.  
Access is enforced strictly through **permissions**, not URL structure.

---

### Endpoint
```
/api/v1/users
```

This endpoint represents the User resource and is primarily used by admins for user management operations.

---

### User APIs (Admin)

| Role  | Method | Endpoint               | Description                         |
|------|--------|------------------------|-------------------------------------|
| Admin | GET    | `/api/v1/users/`        | Retrieve a list of all users.        |
| Admin | GET    | `/api/v1/users/{id}/`   | Retrieve a specific user by ID.      |
| Admin | PATCH  | `/api/v1/users/{id}/`   | Update details of a specific user.   |
| Admin | DELETE | `/api/v1/users/{id}/`   | Delete a user (soft delete).         |

---

### Notes

- These endpoints are **NOT accessible to regular users**.
- Admin access is enforced via:
  - `IsAuthenticated`
  - `IsAdmin` **or** a custom role-based permission check.
- No separate `/admin/users` endpoint is used.
- This design follows the principle:
  **Same resource → same endpoint, different access → permissions**.
- User deletion follows the **soft delete pattern** using a `deleted_at` field.

---

### Permission Enforcement (Conceptual)

permission_classes = [IsAuthenticated, IsAdmin]


## Song Endpoints (Shared Resource)

### Endpoint


```
/api/v1/songs
```

This endpoint is shared by **users and admins**, with access controlled via permissions and queryset filtering.

---

### Song APIs

| Role          | Method | Endpoint             | Description                                    |
| ------------- | ------ | -------------------- | ---------------------------------------------- |
| User / Admin  | GET    | `/api/v1/songs/`      | List songs (user → own, admin → all).          |
| Owner / Admin | GET    | `/api/v1/songs/{id}/` | Retrieve specific song.                        |
| User   | POST   | `/api/v1/songs/`      | Submit new song request (`PENDING`).           |
| Owner  | PATCH    | `/api/v1/songs/{id}/` | Request song update (admin approval required). |
| Owner / Admin | DELETE | `/api/v1/songs/{id}/` | Delete a song (soft delete).                   |

---

### Song Review Endpoint

| Role  | Method | Endpoint                     | Description                                      |
| ----- | ------ | ---------------------------- | ------------------------------------------------ |
| Admin | PATCH  | `/api/v1/songs/{id}/review/` | Approve or reject a song request. |

**Request Body:**

```json
{
  "status": "APPROVED",
  "rejection_reason": "Optional reason"
}
```

**Validation Rules:**

*   `status`: Must be one of `APPROVED`, `REJECTED`.
*   `rejection_reason`: Required if status is `REJECTED`.
*   Only songs with `PENDING` status can be reviewed.

---

### Filtering & Search

Supported query parameters for song listing:

```
/api/v1/songs/?artist=...
/api/v1/songs/?genre=...
/api/v1/songs/?album=...
```

Filters are applied **after** ownership-based queryset restriction.

---

### Queryset Logic (Conceptual)

```python
def get_queryset(self):
    if request.user.is_admin:
        return Song.objects.all()
    return Song.objects.filter(user=request.user)
```

This ensures:

* Users can **never** access other users’ songs
* Admins can access all songs
* Query parameters cannot expand access

---

### Song Lifecycle States

| Status   | Description                                |
| -------- | ------------------------------------------ |
| PENDING  | Awaiting admin review                      |
| APPROVED | Approved and usable                        |
| REJECTED | Rejected by admin (optional reason stored) |

---

## Playlist Endpoints (Shared Resource)

### Endpoint

```
/api/v1/playlists
```

---

### Playlist APIs

| Role               | Method | Endpoint                 | Description                               |
| ------------------ | ------ | ------------------------ | ----------------------------------------- |
| User / Admin       | GET    | `/api/v1/playlists/`      | List playlists (user → own, admin → all). |
| User | POST   | `/api/v1/playlists/`      | Create a new playlist.                    |
| Owner / Admin      | GET    | `/api/v1/playlists/{id}/` | Retrieve playlist details.                |
| Owner              | PATCH    | `/api/v1/playlists/{id}/` | Update playlist information.              |
| Owner / Admin      | DELETE | `/api/v1/playlists/{id}/` | Delete playlist (soft delete).            |


---

### Playlist Song APIs

| Role                   | Method | Endpoint                                    | Description                         |
| ---------------------- | ------ | ------------------------------------------- | ----------------------------------- |
| Owner | POST   | `/api/v1/playlists/{id}/songs/`           | Add an approved song to a playlist. |
| Owner | DELETE | `/api/v1/playlists/{id}/songs/{id}/` | Remove a song from a playlist.      |


### Validation Rules

*   Only `APPROVED` songs can be added.
*   Users can add **any** approved song to their playlists (not restricted to own songs).
*   Duplicate songs in the same playlist are prevented.

---

## Permission Strategy

### Object-Level Permission

```python
class IsOwnerOrAdmin(BasePermission):
    def has_object_permission(self, request, view, obj):
        return request.user.role == UserRole.ADMIN or obj.user == request.user
```

### View-Level Permission

```python
permission_classes = [IsAuthenticated, IsOwnerOrAdmin]
```

### Result

* Users access only their own resources
* Admins bypass ownership restrictions
* No admin-specific endpoints required

---

## Pagination

All list endpoints support pagination via page number pagination.

*   `page`: Page number (default: 1).
*   `page_size`: Number of results per page (default: 10, max: 100).
*   Example: `?page=1&page_size=20`

**Response Format:**
```json
{
  "count": 50,
  "page": 1,
  "page_size": 10,
  "next": 2,
  "previous": null,
  "data": [...]
}
```

## Global Validation Rules

### Field Level
*   **Username**: Not empty or whitespace.
*   **Email**: Must match format `^[a-z0-9._%+-]+@[a-z]+\.[a-z]{2,}$`. Not empty or whitespace.
*   **First Name**: Letters and spaces only. Not empty or whitespace.
*   **Last Name**: Letters and spaces only.
*   **Phone Number**: Digits, spaces, hyphens, and leading plus only.
*   **Password**: Must be at least 8 characters. Used for confirmation in registration.

### Registration
*   **Username**: Unique, case-insensitive.
*   **Email**: Unique, case-insensitive.
*   **confirm_password**: Must match password.

### Song Management
*   **Duration**: Must be a positive integer.
*   **File Upload**: Allowed formats (e.g., mp3) if applicable.

### Playlist Management
*   **Name**: Required, non-empty.

---

## Response Status Codes

All endpoints follow REST best practices:

| Status Code               | Meaning                           |
| ------------------------- | --------------------------------- |
| 200 OK                    | Successful GET, PUT, POST         |
| 201 Created               | Resource successfully created     |
| 204 No Content            | Successful delete or update       |
| 400 Bad Request           | Validation or malformed request   |
| 401 Unauthorized          | Missing or invalid token          |
| 403 Forbidden             | Insufficient permissions          |
| 404 Not Found             | Resource does not exist           |
| 409 Conflict              | Duplicate or constraint violation |
| 500 Internal Server Error | Unexpected server error           |

---


