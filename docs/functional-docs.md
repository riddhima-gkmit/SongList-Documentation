# Functional Documentation

This document explains how the system behaves functionally, focusing on
user roles, access rules, and module responsibilities, strictly aligned
with the defined API design.

---

## User Roles

The system supports two roles:

### USER
- Can register and authenticate
- Can manage **only their own data**
- Can:
  - view, update, and delete their own profile
  - create song requests
  - request updates to songs
  - create and manage their own playlists
- Cannot:
  - access other users’ data
  - approve or reject songs
  - manage other users

---

### ADMIN
- Uses the **same API endpoints** as users
- Has elevated permissions
- Can:
  - view all users
  - update any user
  - delete any user
  - view all songs and playlists
  - approve or reject song creation requests
  - approve or reject song update requests
  - delete any song or playlist

> No role-specific URLs are introduced.  
> All admin capabilities are enforced through **permissions and queryset filtering**.

---

## Core Functional Rules

### Single Endpoint Rule
- Each resource has one canonical endpoint
- No separate admin endpoints are created

Examples:
- `/songs` → used by both users and admins
- `/playlists` → used by both users and admins
- `/users` → used by both users and admins

---

### Access Control Rule
- Differences in access are handled using:
  - role-based permissions
  - object-level ownership checks
  - queryset filtering

---

## System Modules Overview

### Authentication Module

**Purpose**  
Handles user authentication and session management using JWT.

**Key Features**
- User registration
- Login and logout
- Access token refresh
- Password change

---

### User Module

**Purpose**  
Manages user profiles and administrative user operations.

**Key Features**
- Self-service user profile management
- Admin-level user management
- Role-based access enforcement

---

### User Access Rules

| Role | Capability |
|----|-----------|
| User | View, update, delete **own profile only** |
| Admin | View, update, delete **any user** |

**Design Decisions**
- `/users/me` is used for self-service operations
- Admins access user management via the same resource using permissions
- No `/users/{id}` endpoint is exposed to non-admin users

**Example Endpoints**
- `GET /api/v1/users/me`
- `PUT /api/v1/users/me`
- `DELETE /api/v1/users/me`

(Admin-only access via permissions for full user management)

---

### Song Module

**Purpose**  
Manages the personal music library with an approval-based workflow.

**Key Features**
- Song creation requires admin approval
- Song updates require admin approval
- Song lifecycle states:
  - `PENDING`
  - `APPROVED`
  - `REJECTED`
- Users can view only their own songs
- Admins can view all songs

---

### Song Visibility Rules

| Role | Visible Songs |
|----|----|
| User | Own songs only |
| Admin | All songs |

Filtering is supported using query parameters.
