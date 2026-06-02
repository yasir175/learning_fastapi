# FastAPI Social Posts API

A full-featured RESTful API built with **FastAPI** and **PostgreSQL**, implementing a social-media-style backend where users can register, create posts, and vote on them. The project is deployed on **Render** and uses **Alembic** for database migrations.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Database Schema](#database-schema)
- [API Endpoints](#api-endpoints)
- [Environment Variables](#environment-variables)
- [Getting Started](#getting-started)
- [Database Migrations](#database-migrations)
- [Deployment](#deployment)

---

## Overview

This project is a learning-focused but production-ready FastAPI application that covers the full backend development lifecycle:

- User **registration and authentication** using JWT tokens
- **CRUD operations** for posts
- A **voting system** (like/unlike posts)
- Database integration with **PostgreSQL** via **SQLAlchemy**
- Schema migrations with **Alembic**
- Environment-based configuration using **Pydantic Settings**
- **CORS middleware** enabled for frontend integration
- Deployed on **Render** with a hosted PostgreSQL database

---

## Features

- **JWT Authentication** — Secure login with access tokens (HS256, configurable expiry)
- **User Management** — Register and retrieve user profiles with hashed passwords (bcrypt)
- **Post Management** — Create, read, update, and delete posts tied to authenticated users
- **Vote System** — Users can upvote posts; votes are unique per user/post pair
- **Cascade Deletes** — Deleting a user removes their posts and votes automatically
- **CORS Support** — Open to all origins for easy frontend integration
- **Auto-generated API Docs** — Available at `/docs` (Swagger UI) and `/redoc`
- **Database Health Check** — `/db-test` endpoint to verify connectivity

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI 0.116.1 |
| Language | Python 3.x |
| Database | PostgreSQL (hosted on Render) |
| ORM | SQLAlchemy 2.0 |
| Migrations | Alembic 1.16.4 |
| Validation | Pydantic v2 + pydantic-settings |
| Auth | python-jose (JWT) + passlib (bcrypt) |
| DB Driver | psycopg 3 (psycopg-binary) |
| Server | Uvicorn |
| Deployment | Render |

---

## Project Structure

```
fastapi/
├── app/
│   ├── main.py           # FastAPI app entry point, middleware, route registration
│   ├── models.py         # SQLAlchemy ORM models (User, Post, Vote)
│   ├── schemas.py        # Pydantic schemas for request/response validation
│   ├── database.py       # Database engine and session setup
│   ├── config.py         # Pydantic Settings (loads from .env)
│   ├── oauth2.py         # JWT token creation and verification
│   ├── utils.py          # Password hashing utilities
│   └── routers/
│       ├── post.py       # Post CRUD endpoints
│       ├── user.py       # User registration/profile endpoints
│       ├── auth.py       # Login / token generation
│       └── vote.py       # Vote (upvote/unvote) endpoints
├── alembic/
│   ├── env.py            # Alembic environment config (reads from app settings)
│   ├── script.py.mako    # Migration file template
│   └── versions/         # Migration revision files
├── alembic.ini           # Alembic configuration file
├── requirements.txt      # All Python dependencies
└── .env                  # Environment variables (not committed)
```

---

## Database Schema

### `users`
| Column | Type | Notes |
|---|---|---|
| id | Integer | Primary key, auto-increment |
| email | String | Unique, not null |
| password | String | Bcrypt hashed |
| created_at | Timestamp | Server default: now() |

### `posts`
| Column | Type | Notes |
|---|---|---|
| id | Integer | Primary key, auto-increment |
| title | String | Not null |
| content | String | Not null |
| published | Boolean | Default: True |
| created_at | Timestamp | Server default: now() |
| user_id | Integer | FK → users.id (CASCADE delete) |

### `votes`
| Column | Type | Notes |
|---|---|---|
| user_id | Integer | FK → users.id (CASCADE delete) |
| post_id | Integer | FK → posts.id (CASCADE delete) |
| *(user_id, post_id)* | — | Composite primary key (one vote per user per post) |

---

## API Endpoints

### Root
| Method | Path | Description |
|---|---|---|
| GET | `/` | Health check — returns `{"message": "Hello"}` |
| GET | `/db-test` | Tests database connectivity |

### Auth — `/auth`
| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/login` | Log in with email & password, returns JWT access token | ❌ |

### Users — `/users`
| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/` | Register a new user | ❌ |
| GET | `/{id}` | Get a user's profile by ID | ❌ |

### Posts — `/posts`
| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/` | List all posts (with vote counts) | ❌ |
| POST | `/` | Create a new post | ✅ |
| GET | `/{id}` | Get a single post by ID | ❌ |
| PUT | `/{id}` | Update a post (owner only) | ✅ |
| DELETE | `/{id}` | Delete a post (owner only) | ✅ |

### Votes — `/vote`
| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/` | Upvote or unvote a post | ✅ |

> **Note:** Interactive docs available at `http://localhost:8000/docs` after running locally.

---

## Environment Variables

Create a `.env` file in the project root with the following variables:

```env
DATABASE_HOSTNAME=your_postgres_host
DATABASE_PORT=5432
DATABASE_PASSWORD=your_password
DATABASE_NAME=your_db_name
DATABASE_USERNAME=your_username

SECRET_KEY=your_jwt_secret_key
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60

DATABASE_URL=postgresql+psycopg://user:password@host:5432/dbname?sslmode=require
```
## Getting Started

### Prerequisites

- Python 3.10+
- PostgreSQL database (local or hosted)

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/yasir175/learning_fastapi.git
cd learning_fastapi

# 2. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Create your .env file (see Environment Variables section above)

# 5. Run database migrations
alembic upgrade head

# 6. Start the development server
uvicorn app.main:app --reload
```

The API will be available at `http://localhost:8000`.

---

## Database Migrations

This project uses **Alembic** for schema migrations. The `alembic/env.py` is configured to read database credentials from your `.env` file via `app.config`.

```bash
# Apply all pending migrations
alembic upgrade head

# Create a new migration (after changing models.py)
alembic revision --autogenerate -m "describe your change"

# Downgrade by one revision
alembic downgrade -1

# View migration history
alembic history
```

Migration files are stored in `alembic/versions/`. The current schema includes the full `posts`, `users`, and `votes` tables with all foreign key constraints and cascade rules.

---

## Deployment

The application is deployed on **[Render](https://render.com)**:

- **Web Service**: Runs the FastAPI app via `uvicorn`
- **PostgreSQL**: Hosted Render PostgreSQL database (Singapore region)
- **SSL**: Enforced via `sslmode=require` in the connection string

The `DATABASE_URL` environment variable (set in the Render dashboard) is used by both SQLAlchemy and Alembic for the production database connection.

---

## Key Dependencies

```
fastapi==0.116.1
uvicorn==0.35.0
sqlalchemy==2.0.43
alembic==1.16.4
psycopg==3.2.9
pydantic==2.11.7
pydantic-settings==2.10.1
python-jose==3.5.0
passlib==1.7.4
bcrypt==4.0.1
python-dotenv==1.1.1
email_validator==2.2.0
```

Full list available in [`requirements.txt`](./requirements.txt).

---

## Author

**Yasir** — [@yasir175](https://github.com/yasir175)

---

*Built while learning FastAPI — covering authentication, database integration, migrations, and cloud deployment from scratch.*
