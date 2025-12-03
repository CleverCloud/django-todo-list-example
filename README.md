![Clever Cloud logo](/assets/clever-cloud-logo.png)

# Django Todo List Example for Clever Cloud

[![Clever Cloud - PaaS](https://img.shields.io/badge/Clever%20Cloud-PaaS-orange?logo=clevercloud)](https://clever-cloud.com)
[![Django](https://img.shields.io/badge/Django-5.2-092E20?logo=django&logoColor=white)](https://www.djangoproject.com/)
[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?logo=python&logoColor=white)](https://www.python.org/)

A simple Django todo list application using modern Python tooling (uv, pyproject.toml) that deploys easily on Clever Cloud.

![Screenshot](/assets/screenshot.jpg)

## Features

- Create multiple todo lists
- Add, complete, and delete todo items
- Simple, clean UI
- PostgreSQL database support (automatically configured on Clever Cloud)
- SQLite fallback for local development (no database setup required)
- Modern Python tooling with uv

## Local Development

### Prerequisites

- Python 3.11+
- [uv](https://github.com/astral-sh/uv) installed

### Setup

1. Clone the repository and navigate to the project directory

2. Install dependencies:
   ```bash
   uv sync
   ```

3. Run migrations:
   ```bash
   uv run python manage.py makemigrations
   uv run python manage.py migrate
   ```

4. Start the development server:
   ```bash
   uv run python manage.py runserver
   ```

5. Visit http://127.0.0.1:8000/ in your browser

### Local Configuration

Create a `.env` file for local development (optional):
```bash
DEBUG=True
SECRET_KEY=your-secret-key-here
ALLOWED_HOSTS=localhost,127.0.0.1
```

**Database Configuration:**
- **Local development**: Uses SQLite by default (no configuration needed)
- **Production**: Automatically uses PostgreSQL when `POSTGRESQL_ADDON_HOST` is set
- The app detects the environment and switches databases automatically

## Deployment on Clever Cloud

### Step 1: Create a Python Application

```bash
# Install Clever Cloud CLI if you haven't already
# See: https://www.clever.cloud/developers/doc/cli/

# Login
clever login

# Create a Python application
clever create --type python django-todo-list
```

### Step 2: Add a PostgreSQL Database

```bash
# Create a PostgreSQL addon
clever addon create postgresql-addon my-todo-db

# Link it to your application
clever service link-addon my-todo-db
```

This automatically sets the `POSTGRESQL_ADDON_*` environment variables that the app uses.

### Step 3: Configure Environment Variables

Set the required environment variables:

```bash
# Required: Django secret key (generate a secure one!)
clever env set SECRET_KEY "your-super-secret-key-here"

# Required: Your domain (or use the default .cleverapps.io domain)
clever env set ALLOWED_HOSTS "your-app.cleverapps.io"

# Required: Use native uv support with locked dependencies
clever env set CC_PYTHON_UV_SYNC_FLAGS "--frozen"

# Required: Run the application with uvicorn on port 8080 (native uv support)
clever env set CC_PYTHON_UV_RUN_COMMAND ".venv/bin/uvicorn config.asgi:application --host 0.0.0.0 --port 8080"

# Required: Run migrations and collect static files on deployment
clever env set CC_PRE_RUN_HOOK "uv run python manage.py migrate --noinput && uv run python manage.py collectstatic --noinput"
```

### Step 4: Deploy

```bash
clever deploy
```

### Environment Variables Reference

| Variable | Required | Description | Default |
|----------|----------|-------------|---------|
| `SECRET_KEY` | Yes | Django secret key | Generated insecure key (dev only) |
| `ALLOWED_HOSTS` | Yes | Comma-separated list of allowed hosts | `localhost,127.0.0.1` |
| `CC_PYTHON_UV_SYNC_FLAGS` | Yes | uv sync flags for dependency installation | `--frozen` |
| `CC_PYTHON_UV_RUN_COMMAND` | Yes | Command to run with native uv support | - |
| `CC_PRE_RUN_HOOK` | Yes | Commands to run before starting | - |
| `DEBUG` | No | Enable Django debug mode | `False` |
| `POSTGRESQL_ADDON_HOST` | Auto | PostgreSQL host (set by Clever Cloud addon) | - |
| `POSTGRESQL_ADDON_PORT` | Auto | PostgreSQL port (set by Clever Cloud addon) | `5432` |
| `POSTGRESQL_ADDON_DB` | Auto | PostgreSQL database name (set by Clever Cloud addon) | `todolist` |
| `POSTGRESQL_ADDON_USER` | Auto | PostgreSQL username (set by Clever Cloud addon) | `postgres` |
| `POSTGRESQL_ADDON_PASSWORD` | Auto | PostgreSQL password (set by Clever Cloud addon) | - |

### Static Files

Static files are collected to the `staticfiles/` directory during deployment via the `CC_PRE_RUN_HOOK`.

## Project Structure

```
.
├── config/              # Django project settings
│   ├── settings.py      # Main settings (environment-aware)
│   ├── urls.py          # URL routing
│   ├── asgi.py          # ASGI entry point
│   └── wsgi.py          # WSGI entry point (legacy)
├── todos/               # Todo app
│   ├── models.py        # TodoList and TodoItem models
│   ├── views.py         # CRUD views
│   ├── urls.py          # App URLs
│   └── templates/       # HTML templates
├── manage.py            # Django management script
├── pyproject.toml       # Python dependencies (uv)
└── README.md           # This file
```

## Database Models

### TodoList
- `title`: List name
- `created_at`: Creation timestamp
- `updated_at`: Last update timestamp

### TodoItem
- `todo_list`: Foreign key to TodoList
- `title`: Item description
- `completed`: Boolean completion status
- `created_at`: Creation timestamp
- `updated_at`: Last update timestamp

## API Endpoints

| URL | Method | Description |
|-----|--------|-------------|
| `/` | GET | List all todo lists |
| `/lists/create/` | POST | Create a new list |
| `/lists/<id>/` | GET | View a specific list and its items |
| `/lists/<id>/delete/` | POST | Delete a list |
| `/lists/<id>/items/create/` | POST | Add an item to a list |
| `/lists/<id>/items/<item_id>/toggle/` | POST | Toggle item completion |
| `/lists/<id>/items/<item_id>/delete/` | POST | Delete an item |

## Technologies

- **Django 5.2**: Web framework with ASGI support
- **uv**: Fast Python package installer and resolver
- **Uvicorn**: Modern ASGI server
- **PostgreSQL**: Production database (via Clever Cloud addon)
- **SQLite**: Local development database

## License

MIT
