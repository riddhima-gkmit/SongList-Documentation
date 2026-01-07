
# SongList

## Personalized Music Library & Playlist Management

---

## Project Overview

**SongList** is a REST-based backend application built using **Django** and **Django REST Framework (DRF)**.

It allows authenticated users to maintain a **personal music library** and create **custom playlists**, while ensuring that all data is securely **isolated per user**.

The system also introduces an **admin approval workflow** to validate song creation and updates before they become publicly usable.

---

## Problem Statement

Music listeners often curate personal song collections across multiple platforms.

However, most existing systems lack one or more of the following:

* Fine-grained access control
* Clean, REST-based APIs for secure personalization
* Moderation workflows for shared content

There is a need for a backend system that:

* Ensures users can manage **only their own data**
* Enforces **approval-based moderation**
* Follows **REST principles** and **Django best practices**

---

## Proposed Solution

SongList addresses these challenges by providing:

* **JWT-based authentication**
* **Role-based access control** (`USER` and `ADMIN`)
* **Moderated song creation and updates**
* **Playlist management using approved songs only**
* **Clean, REST-pure API endpoints**

The system avoids duplicated or role-specific endpoints by relying on:

* Permission classes
* Queryset filtering
* Object-level authorization

This approach keeps the API surface minimal and maintainable.

---

## Objectives

The primary objectives of the SongList system are:

* Implement a complete **authentication and authorization flow**
* Enable full **CRUD operations** for songs and playlists
* Allow users to **organize songs into playlists**
* Enforce **admin approval** for song creation and updates
* Use **enums, constants, and environment variables** consistently
* Follow **clean Django project structure and conventions**

---

