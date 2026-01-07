# Functional Documentation

This document explains how the **SongList system behaves functionally**, focusing on:

* User roles
* Access rules
* Module responsibilities

All behavior described here is **strictly aligned with the defined API design** and implementation.

---

## User Roles

The system supports two roles:

---

### USER

A regular authenticated user with access limited to their own data.

**Capabilities**

* Can register and authenticate
* Can manage **only their own data**

**Can**

* View, update, and delete their own profile
* Create song requests
* Request updates to their own songs
* Create and manage their own playlists

**Cannot**

* Access other users’ data
* Approve or reject songs
* Manage other users

---

### ADMIN

An elevated user role with moderation and management responsibilities.

**Important Note**
Admins use the **same API endpoints** as regular users.
No role-specific URLs are introduced.

**Can**

* View all users
* Update any user
* Delete any user (**except themselves**)
* View all songs and playlists
* Approve or reject song creation/update requests via a review endpoint
* View and delete any playlist
* Delete any song or playlist

**Cannot**

* Create song requests themselves
* Update or add/remove songs in playlists

> Admin privileges are enforced exclusively through **permissions and queryset filtering**, not separate endpoints.

---

## Core Functional Rules

### Single Endpoint Rule

Each resource has **one canonical endpoint**, shared by users and admins.

**Example Endpoints**

```http
/api/v1/songs/
/api/v1/playlists/
/api/v1/users/
```

No duplicated or role-specific endpoints exist.

---

### Access Control Rule

Differences in behavior are handled using:

* Role-based permissions
* Object-level ownership checks
* Queryset filtering

This keeps the API clean, consistent, and REST-pure.

---

## System Modules Overview

### Authentication Module

**Purpose**
Handles user authentication and session management using JWT.

**Key Features**

* User registration
* Login and logout
* Access token refresh
* Password change
* Strict input validation (email format, names, phone numbers)

---

### User Module

**Purpose**
Manages user profiles and administrative user operations.

**Key Features**

* Self-service user profile management
* Admin-level user management
* Role-based access enforcement

---

## User Access Rules

| Role  | Capability                                |
| ----- | ----------------------------------------- |
| User  | View, update, delete **own profile only** |
| Admin | View, update, delete **any user**         |

**Design Decisions**

* `/users/me` is used for self-service operations
* Admins access all users through the same resource using permissions
* No `/users/{id}` endpoint is exposed to non-admin users

**Example Endpoints**

```http
GET /api/v1/users/me/
PUT /api/v1/users/me/
DELETE /api/v1/users/me/
```
(Admin-only access for full user management is enforced via permissions.)

---

## Song Module

**Purpose**
Manages the personal music library with an approval-based lifecycle.

**Key Features**

* Song creation requests start in `PENDING` state
* **Strict ownership**: users can update or delete only songs they created
* Song lifecycle states:

  * `PENDING`
  * `APPROVED`
  * `REJECTED`

---

### Song Visibility Rules

| Role  | Visible Songs                                           |
| ----- | ------------------------------------------------------- |
| User  | All `APPROVED` songs + own `PENDING` / `REJECTED` songs |
| Admin | All songs (any status)                                  |

**Filtering**

* Songs can be filtered by:

  * `artist`
  * `genre`
  * `album`
* Users may filter by `status` (e.g., `status=PENDING`) to view their own requests

---

### Song Review Process

Admins manage song approval using a **dedicated review endpoint**.

**Endpoint**

```http
PATCH /api/v1/songs/{id}/review
```

**Rules**

* Only songs in `PENDING` state can be reviewed
* Admin must provide a status:

  * `APPROVED`
  * `REJECTED`
* If `REJECTED`, a justification reason must be provided

---

## Playlist Module

**Purpose**
Enables users to create and manage personal playlists with curated song collections.

**Key Features**

* Create, update, and delete personal playlists
* Add approved songs from any user to playlists
* Remove songs from playlists
* Soft delete support

---

### Playlist Visibility Rules

| Role  | Visible Playlists           |
| ----- | --------------------------- |
| User  | Only their own playlists    |
| Admin | All playlists               |

---

### Playlist Song Rules

* Only `APPROVED` songs can be added to playlists
* Users can add **any** approved song (not restricted to their own songs)
* Duplicate songs in the same playlist are prevented
* Admins can view and delete playlists but **cannot** modify them (create, update, or add/remove songs)

---

## Summary

The functional design ensures:

* Clean separation of responsibilities
* Strong ownership and access guarantees
* Moderated content lifecycle
* A REST-pure, maintainable API surface

This approach balances **security**, **simplicity**, and **scalability**.

---

