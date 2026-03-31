# Gira

A Jira-inspired project management backend built with FastAPI. Features real-time WebSocket notifications via Redis Streams, async email delivery with Celery, role-based access control, and a built-in admin panel.

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI |
| Language | Python 3.12 |
| Database | PostgreSQL + SQLAlchemy 2.0 |
| Migrations | Alembic |
| Auth | JWT (HS256) + Argon2 password hashing |
| Real-time | WebSockets + Redis Streams |
| Task Queue | Celery + Redis |
| Admin | Starlette Admin |
| Linting | Ruff + pre-commit |

## Features

- **Project & Task Management** -- Create projects, assign tasks with priorities (low/medium/high), track status through a workflow (TODO -> In Progress -> Ready for Test -> Done)
- **Role-Based Access** -- Five roles (admin, manager, developer, tester, user) with granular permissions per endpoint
- **Real-Time Notifications** -- WebSocket connections with Redis Streams consumer groups for multi-worker fan-out. Events include task creation, status changes, test-ready signals, and rejections
- **Async Email** -- Registration confirmation emails sent via Celery workers through SMTP
- **Admin Panel** -- Full CRUD interface at `/admin/` with CSV/Excel/PDF export support
- **Audit Logging** -- Tracks user actions on tasks with timestamps
- **Avatar Uploads** -- Profile image upload with file type/size validation (5MB, jpg/png)

## Project Structure

```
app/
├── admin/           # Starlette Admin panel views & auth
├── routers/         # API route handlers
│   ├── auth.py      # Register, login, email confirmation
│   ├── projects.py  # Project CRUD, member management
│   ├── tasks.py     # Task CRUD, status transitions
│   ├── comments.py  # Task comments
│   ├── notifications.py
│   └── users.py     # Profile, avatar upload
├── schemas/         # Pydantic request/response models
├── services/        # Business logic (key generation, file handling)
├── websocket/       # WS connection manager, Redis Streams pub/sub
├── models.py        # SQLAlchemy ORM models
├── dependencies.py  # Auth & DB dependency injection
├── enums.py         # Role, Status, Priority, Event enums
├── celery.py        # Celery worker for async emails
├── settings.py      # Environment-based configuration
└── main.py          # App entrypoint
```

## API Overview

| Group | Endpoints |
|---|---|
| **Auth** | `POST /auth/register/`, `POST /auth/login/`, `GET /auth/confirm/{token}/` |
| **Users** | `GET /users/profile/`, `PUT /users/profile/update/`, `POST /users/avatar/upload/`, `DELETE /users/profile/delete/` |
| **Projects** | `GET /projects/{key}/`, `POST /projects/create/`, `PUT /projects/{key}/update/`, `POST /projects/{key}/members/invite/`, `POST /projects/{key}/members/kick/` |
| **Tasks** | `GET /tasks/{key}/`, `POST /tasks/create/`, `PUT /tasks/{id}/update/`, `PATCH /tasks/{id}/move/`, `DELETE /tasks/{id}/delete/` |
| **Comments** | `POST /comments/create/`, `PUT /comments/{id}/update/`, `DELETE /comments/{id}/delete/` |
| **Notifications** | `GET /notifications/`, `PUT /notifications/{id}/` |
| **WebSocket** | `WS /ws/connect?token={jwt}` |
| **Admin** | `GET /admin/` |

## Getting Started

### Prerequisites

- Python 3.12+
- PostgreSQL
- Redis

### Setup

```bash
# Clone
git clone git@github.com:Samandar-Komilov/gira.git
cd gira

# Environment
cp .env.example .env
# Edit .env -- set DB credentials and SECRET_KEY

# Install dependencies
uv sync

# Run migrations
alembic upgrade head

# Start the app
uvicorn app.main:app --reload
```

### Running Celery (for async emails)

```bash
# Worker
celery -A app.celery worker --loglevel=info

# Monitor (optional)
celery -A app.celery flower
```

### Environment Variables

```
PROJECT_NAME=Agile
DB_USER=postgres
DB_PASSWORD=<your_password>
DB_HOST=localhost
DB_PORT=5432
DB_NAME=agile_db
SECRET_KEY=<your_secret_key>
```

## WebSocket Events

Clients connect via `ws://host/ws/connect?token=<jwt>` and receive events based on their role:

| Event | Triggered When | Audience |
|---|---|---|
| `task_created` | New task created | Project members |
| `task_created_high` | High-priority task created | All roles |
| `task_status_change` | Task status updated | Managers |
| `task_move_ready` | Task moved to Ready for Test | Testers |
| `task_rejected` | Task rejected | Developers |

Multi-worker delivery is handled through Redis Streams with consumer groups, ensuring each connected client receives the event exactly once regardless of which worker they're connected to.

## License

MIT
