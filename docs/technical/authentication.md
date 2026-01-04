# Authentication & Authorization

SongList uses **RBAC (Role-Based Access Control)** combined with **object-level permissions** to ensure users can only perform actions allowed for their role and ownership.

---

## Authentication Flow

### 1. User Registration

* User registers using **username, email, and password**.
* Role is assigned as **USER** by default.
* Passwords are securely hashed using Django’s authentication system.

### 2. User Login

* User logs in with valid credentials.
* System issues **JWT access and refresh tokens**.
* Access token is required for all protected APIs.

### 3. Token Refresh

* Refresh token is used to obtain a new access token.
* Prevents frequent re-authentication.

### 4. Password Management

* Authenticated users can change their password.
* Logout invalidates refresh tokens.

### 5. Authentication Enforcement

* All non-auth APIs require a valid JWT.
* Unauthenticated access returns **401 Unauthorized**.

---

## Authorization Rules (Role-Based Access Control)

### Role Capabilities Matrix

| Action                    | User            | Admin                              |
| ------------------------- | --------------- | ---------------------------------- |
| **Authentication**        |                 |                                    |
| Register / Login          | ✅               | ✅                                  |
| Refresh Token             | ✅               | ✅                                  |
| Change Password           | ✅               | ✅                                  |
| Logout                    | ✅               | ✅                                  |
| **User Management**       |                 |                                    |
| View Own Profile          | ✅               | ✅                                  |
| Update Own Profile        | ✅               | ✅                                  |
| Delete Own Profile        | ✅               | ✅                                  |
| View All Users            | ❌               | ✅                                  |
| View Any User             | ❌               | ✅                                  |
| Update Any User           | ❌               | ✅                                  |
| Delete Any User           | ❌               | ✅                                  |
| **Song Management**       |                 |                                    |
| Create Song Request       | ✅               | ❌ *(Admin can directly add songs)* |
| View Songs                | ✅ *(own)*       | ✅ *(all)*                          |
| View Song Details         | ✅ *(own)*       | ✅ *(all)*                          |
| Request Song Update       | ✅ *(own)*       | ❌ *(Admin updates directly)*       |
| Delete Song               | ✅ *(own)*       | ✅                                  |
| Approve / Reject Songs    | ❌               | ✅                                  |
| **Playlist Management**   |                 |                                    |
| Create Playlist           | ✅               | ❌                                  |
| View Playlists            | ✅ *(own)*       | ✅ *(all)*                          |
| Update Playlist           | ✅ *(own)*       | ❌                                  |
| Delete Playlist           | ✅ *(own)*       | ❌                                  |
| Add Song to Playlist      | ✅ *(own songs)* | ❌                                  |
| Remove Song from Playlist | ✅ *(own songs)* | ❌                                  |

---

## Key Authorization Rules

### Songs

* Users can access **only their own songs**.
* Admins can access **all songs**.
* All song creation and update requests require **admin approval**.
* Rejected updates do **not overwrite** approved data.

### Playlists

* Playlists are **owned by users**.
* Users can modify **only their own playlists**.
* Admins can view **any playlist**.
* Only **approved songs owned by the same user** can be added to a playlist.

### Users

* Users can manage **only their own profiles**.
* Admins can manage **all users**.
* No role-specific endpoints are exposed.

---

## Permission Enforcement

### Permission Classes Used

- **AllowAny** – Public endpoints (register, login, token refresh)
- **IsAuthenticated** – All protected APIs
- **IsAdminUser** – Admin-only APIs (user management, approvals)
- **IsOwnerOrAdmin** – Object-level permission for owned resources


```python
class IsOwnerOrAdmin(BasePermission):
    def has_object_permission(self, request, view, obj):
        return request.user.is_staff or obj.user == request.user
```

---

## Authorization Strategy

* **View-level permissions** restrict access to authenticated users.
* **Queryset filtering** enforces role-based data visibility.
* **Object-level permissions** enforce ownership rules.
* No admin-specific URLs or role-based routing is used.

---

## Summary

* **Users** → Manage personal songs and playlists with admin-moderated approval.
* **Admins** → Manage all users, songs, and playlists, and approve or reject song requests.
* **System** → Enforces security using JWT, RBAC, queryset filtering, and object-level permissions.

---

