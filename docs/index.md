# SongList – Personalized Music Library & Playlist Management

## Project Overview
SongList is a REST-based backend application built using **Django** and **Django REST Framework**.  
It allows authenticated users to maintain a **personal music library** and create **custom playlists**, while ensuring that all data is securely isolated per user.

The system also introduces an **admin approval workflow** to validate song creation and updates before they become publicly usable.

---

## Problem Statement
Music listeners often curate personal song collections across platforms.  
However, most systems either lack:
- fine-grained access control, or
- clean REST-based APIs for secure personalization.

There is a need for a backend system that:
- ensures users can manage only their own data,
- enforces approval-based moderation,
- follows REST and Django best practices.

---

## Proposed Solution
SongList solves this problem by providing:
- JWT-based authentication
- Role-based access control (User & Admin)
- Moderated song creation and updates
- Playlist management with approved songs only
- Clean, REST-pure API endpoints

The system avoids duplicated endpoints by using **permissions and queryset filtering** instead of role-specific URLs.

---

## Objectives
- Implement a complete authentication flow
- Enable CRUD operations for songs and playlists
- Allow users to organize songs into playlists
- Enforce admin approval for song changes
- Use enums, constants, and environment variables
- Follow clean Django project structuring
