# Database Schema

## Data Models Overview

The database schema is designed to ensure:

* **Data isolation** – users can access only their own songs and playlists
* **Referential integrity** – all relationships are enforced using foreign keys
* **Easy extensibility** – new features can be added without major schema changes

The core entities in the system are:

* **User** – represents authenticated system users
* **Genre** – categorizes songs
* **Song** – stores music library data with approval workflow
* **Playlist** – groups songs for users
* **PlaylistSong** – manages the many-to-many relationship between playlists and songs

---

## Tables and Fields

### User

Stores authentication credentials, profile information, and role details.

| Field       | Type                 | Description                     |
| ----------- | -------------------- | ------------------------------- |
| id          | Integer (PK)         | Unique identifier for the user  |
| role        | Enum                 | Defines role: `ADMIN` or `USER` |
| username    | String               | Unique username                 |
| email       | String               | User email address              |
| password    | String               | Hashed password                 |
| first_name  | String               | User’s first name               |
| last_name   | String               | User’s last name                |
| phone_no    | String               | Contact phone number            |
| is_active   | Boolean              | Indicates if account is active  |
| date_joined | Timestamp            | Account creation timestamp      |
| updated_at  | Timestamp            | Last profile update             |
| deleted_at  | Timestamp (nullable) | Soft delete timestamp           |

---

### Genre

Represents song categorization to support filtering and organization.

| Field       | Type                 | Description                   |
| ----------- | -------------------- | ----------------------------- |
| id          | Integer (PK)         | Unique genre identifier       |
| name        | String               | Genre name (e.g., Rock, Jazz) |
| description | String               | Optional genre description    |
| created_at  | Timestamp            | Record creation time          |
| updated_at  | Timestamp            | Last update time              |

---

### Song

Stores song metadata and enforces an approval-based lifecycle.

| Field            | Type                 | Description                       |
| ---------------- | -------------------- | --------------------------------- |
| id               | Integer (PK)         | Unique song identifier            |
| user_id          | Integer (FK)         | Owner of the song                 |
| genre_id         | Integer (FK)         | Associated genre                  |
| status           | Enum                 | `PENDING`, `APPROVED`, `REJECTED` |
| rejection_reason | String (nullable)    | Reason for rejection (if any)     |
| title            | String               | Song title                        |
| artist           | String               | Artist name                       |
| album            | String (nullable)    | Album name                        |
| duration         | SmallInt             | Duration in seconds               |
| release_year     | SmallInt             | Year of release                   |
| created_at       | Timestamp            | Song creation time                |
| updated_at       | Timestamp            | Last modification time            |
| deleted_at       | Timestamp (nullable) | Soft delete timestamp             |

**Key Notes**

* Users cannot modify song data without admin approval.
* Only approved songs can be added to playlists.
* Status field enables moderation workflow.

---

### Playlist

Represents a user-defined collection of songs.

| Field       | Type                 | Description                    |
| ----------- | -------------------- | ------------------------------ |
| id          | Integer (PK)         | Unique playlist identifier     |
| user_id     | Integer (FK)         | Playlist owner                 |
| name        | String               | Playlist name                  |
| description | String               | Playlist description           |
| created_at  | Timestamp            | Creation timestamp             |
| updated_at  | Timestamp            | Last update time               |
| deleted_at  | Timestamp (nullable) | Soft delete timestamp          |

**Key Notes**

* Ownership determines access permissions.

---

### PlaylistSong

Join table enabling a many-to-many relationship between playlists and songs.

| Field       | Type                 | Description               |
| ----------- | -------------------- | ------------------------- |
| id          | Integer (PK)         | Unique mapping identifier |
| playlist_id | Integer (FK)         | Related playlist          |
| song_id     | Integer (FK)         | Related song              |
| created_at  | Timestamp            | Time of association       |
| updated_at  | Timestamp            | Last update time          |
| deleted_at  | Timestamp (nullable) | Soft delete timestamp     |

**Key Notes**

* Prevents duplicate song entries in a playlist.
* Ensures playlists contain only approved songs.

---

## ER Diagram

![SongList Schema Design](../assets/images/SongList-Schema-Design.png)


---

## Design Considerations

* **Soft Delete Pattern** ensures data safety and auditability
* **Enums** prevent invalid role and status values
* **Foreign Keys** maintain strict ownership and relationships

---

## Summary

This schema supports:

* Secure, user-isolated data access
* Moderated song lifecycle management
* Efficient playlist-song relationships
* Clean REST API integration

The design balances **simplicity**, **scalability**, and **production-readiness**.

---

