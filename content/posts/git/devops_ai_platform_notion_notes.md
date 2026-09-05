# DevOps AI Platform — Backend Notes (Up to User Creation)

## 1. Project Goal

We are building a DevOps AI Platform: a single dashboard that will eventually bring multiple DevOps capabilities/tools into one place.

Current backend foundation:
- FastAPI
- Pydantic / Pydantic Settings
- SQLAlchemy
- PostgreSQL
- Alembic
- Uvicorn
- Swagger/OpenAPI

---

## 2. High-Level Architecture

### Runtime request/data flow

```text
Swagger UI / Frontend
        |
        | HTTP request
        v
     FastAPI
        |
        v
   Route Layer
        |
        v
 Pydantic Schema
   Validation
        |
        v
  Service Layer
 Business Logic
        |
        v
SQLAlchemy Model
        |
        v
SQLAlchemy Session
        |
        v
  PostgreSQL
        |
        v
   Database
```

### Separate database-schema flow

```text
SQLAlchemy Models
       |
       v
    Alembic
       |
       v
Migration File
       |
       v
PostgreSQL Schema
```

**Important:** Alembic manages database structure/schema changes. It is not normally involved in every API request.

---

# 3. Project Structure

```text
backend/
├── .env
├── requirements.txt
├── alembic.ini
│
├── alembic/
│   ├── env.py
│   └── versions/
│       └── <migration files>
│
└── app/
    ├── main.py
    ├── config/
    │   └── settings.py
    ├── database/
    │   ├── base.py
    │   └── session.py
    ├── models/
    │   └── user.py
    ├── schemas/
    │   └── user.py
    ├── services/
    │   └── user_service.py
    └── routes/
        ├── health.py
        └── users.py
```

You do not need to memorize every file. Learn the responsibility of each layer.

---

# 4. `.env` — Configuration

File:

```text
backend/.env
```

Contains environment-specific values such as:

```env
APP_NAME=DevOps AI Platform
APP_VERSION=1.0.0
APP_ENV=development

HOST=0.0.0.0
PORT=8000
DEBUG=true

DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=devops_ai_platform
DATABASE_USER=hemanth
DATABASE_PASSWORD=...
```

Why use `.env`?

- Avoid hardcoding configuration
- Keep secrets out of source code
- Easily change values between environments
- Centralize configuration

---

# 5. `settings.py` — Pydantic Settings

File:

```text
app/config/settings.py
```

This defines a `Settings` class using `BaseSettings`.

Conceptually:

```text
.env
  |
  v
Pydantic Settings
  |
  v
settings object
```

Other files can then use:

```python
settings.database_name
settings.database_host
settings.port
settings.app_name
```

Pydantic Settings also converts/validates types. For example:

```env
PORT=8000
```

can become:

```python
port: int
```

---

# 6. PostgreSQL

We installed PostgreSQL 17 and created:

```text
devops_ai_platform
```

database.

Connect:

```bash
psql devops_ai_platform
```

The directory from which you run `psql` does not matter.

Useful commands:

```sql
\l
```

List databases.

```sql
\dt
```

List tables.

```sql
\d users
```

Describe the users table.

```sql
SELECT * FROM users;
```

Read users.

```sql
SELECT COUNT(*) FROM users;
```

Count users.

```sql
\q
```

Exit.

### `psql` prompt

```text
devops_ai_platform=#
```

means PostgreSQL is ready for a new command.

```text
devops_ai_platform-#
```

means PostgreSQL is waiting for the current SQL statement to finish.

Usually you forgot the semicolon:

```sql
SELECT * FROM users;
```

If stuck, press `Ctrl+C`.

---

# 7. SQLAlchemy — ORM

SQLAlchemy is the Python SQL toolkit/ORM.

ORM means:

> Object-Relational Mapping

The basic relationship is:

```text
Python Class       <-> Database Table
Python Object      <-> Database Row
Python Attribute   <-> Database Column
```

Example:

```python
class User(Base):
    __tablename__ = "users"
```

represents the `users` database table.

---

# 8. `database/base.py` — SQLAlchemy Base

File:

```text
app/database/base.py
```

We created:

```python
from sqlalchemy.orm import DeclarativeBase

class Base(DeclarativeBase):
    pass
```

Our SQLAlchemy models inherit from `Base`:

```python
class User(Base):
```

`Base.metadata` contains information about the application's SQLAlchemy tables.

Alembic uses this metadata for autogeneration.

---

# 9. `models/user.py` — Database Model

File:

```text
app/models/user.py
```

Our model is conceptually:

```python
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)

    username: Mapped[str] = mapped_column(
        String(100),
        unique=True,
        nullable=False
    )

    email: Mapped[str] = mapped_column(
        String(255),
        unique=True,
        nullable=False
    )
```

This represents the PostgreSQL `users` table.

Conceptually:

```text
User class
    |
    v
users table

id        -> id column
username  -> username column
email     -> email column
```

---

# 10. Alembic — Database Migrations

Alembic manages database schema changes.

Examples:

- Create a table
- Add a column
- Rename a column
- Add an index
- Change a constraint

We used:

```bash
alembic revision --autogenerate -m "create users table"
```

to generate a migration.

Then:

```bash
alembic upgrade head
```

to apply it.

Migration files live under:

```text
alembic/versions/
```

### Alembic flow

```text
SQLAlchemy Model
      |
      v
Base.metadata
      |
      v
Alembic
      |
      v
Migration
      |
      v
PostgreSQL
```

We currently have:

```text
devops_ai_platform
├── alembic_version
└── users
```

`alembic_version` is maintained by Alembic to track applied migrations.

---

# 11. `alembic/env.py`

File:

```text
alembic/env.py
```

This connects Alembic to our application configuration and SQLAlchemy metadata.

It imports:

```python
from app.config.settings import settings
from app.database.base import Base
from app.models.user import User
```

and uses:

```python
target_metadata = Base.metadata
```

This allows Alembic's autogeneration to discover the `users` table from the SQLAlchemy model.

---

# 12. `database/session.py` — Engine and Session

File:

```text
app/database/session.py
```

First, we build the database URL:

```python
DATABASE_URL = (
    f"postgresql+psycopg://"
    f"{settings.database_user}:"
    f"{settings.database_password}@"
    f"{settings.database_host}:"
    f"{settings.database_port}/"
    f"{settings.database_name}"
)
```

Then:

```python
engine = create_engine(DATABASE_URL)
```

### Engine

The SQLAlchemy engine knows how to communicate with PostgreSQL.

```text
DATABASE_URL
      |
      v
SQLAlchemy Engine
      |
      v
PostgreSQL
```

Then we create a session factory:

```python
SessionLocal = sessionmaker(
    autocommit=False,
    autoflush=False,
    bind=engine,
)
```

Think:

```text
SessionLocal()
     |
     v
SQLAlchemy Database Session
```

---

# 13. `get_db()` — Database Dependency

Also in:

```text
app/database/session.py
```

we have:

```python
def get_db():
    db = SessionLocal()

    try:
        yield db
    finally:
        db.close()
```

What it does:

1. Create a database session.
2. Give the session to the request.
3. After the request finishes, close the session.

Conceptually:

```text
Request starts
     |
     v
get_db()
     |
     v
SessionLocal()
     |
     v
Create Session
     |
     v
Give Session to route
     |
     v
Route/service uses it
     |
     v
Request finishes
     |
     v
db.close()
```

**Important:** The service does not execute `session.py`. The `session.py` module defines how sessions are created and managed. FastAPI obtains a session through `get_db()`, and the service uses that session object.

---

# 14. Pydantic Schema

File:

```text
app/schemas/user.py
```

We created a request schema similar to:

```python
from pydantic import BaseModel, EmailStr

class UserCreate(BaseModel):
    username: str
    email: EmailStr
```

This is different from the SQLAlchemy model.

### Pydantic Schema

Purpose:

```text
Validate API input
```

### SQLAlchemy Model

Purpose:

```text
Represent database table/data
```

Remember:

```text
Pydantic
    -> API data validation

SQLAlchemy
    -> Database mapping/operations
```

---

# 15. What Pydantic Actually Validates

Suppose Swagger sends:

```json
{
    "username": "hemanth",
    "email": "hemanth@gmail.com"
}
```

Pydantic checks the request against:

```python
class UserCreate(BaseModel):
    username: str
    email: EmailStr
```

So it checks things such as:

```text
username provided?       Yes
username is a string?    Yes
email provided?          Yes
email has valid format?  Yes
```

Pydantic does **not** query PostgreSQL to ask whether `hemanth` already exists.

That is database/business logic.

---

# 16. `EmailStr` and `email-validator`

When we used:

```python
email: EmailStr
```

Pydantic required the `email-validator` dependency.

We encountered:

```text
ModuleNotFoundError: No module named 'email_validator'
```

This illustrates an important dependency relationship:

```text
EmailStr
   |
   v
email-validator package
```

---

# 17. `routes/users.py` — API Route

File:

```text
app/routes/users.py
```

Conceptually:

```python
@router.post("/users")
def create_new_user(
    user: UserCreate,
    db: Session = Depends(get_db),
):
    return create_user(db, user)
```

The route receives:

```text
user
  |
  +--> Pydantic UserCreate object

db
  |
  +--> SQLAlchemy Session
```

The route is responsible for the HTTP/API layer.

It delegates the actual application logic to the service.

---

# 18. `Depends(get_db)`

This:

```python
db: Session = Depends(get_db)
```

means:

> FastAPI should call `get_db()` and provide the resulting database session here.

It does not mean:

> Execute the entire `session.py` file.

The relationship is:

```text
session.py
    |
    | defines
    v
get_db()
    |
    | creates
    v
SQLAlchemy Session
    |
    | provided to
    v
Route
```

---

# 19. `services/user_service.py` — Business Logic

File:

```text
app/services/user_service.py
```

The service receives both:

```text
db   -> SQLAlchemy Session
user -> validated Pydantic object
```

Conceptually:

```python
def create_user(db: Session, user: UserCreate):

    new_user = User(
        username=user.username,
        email=user.email,
    )

    db.add(new_user)
    db.commit()
    db.refresh(new_user)

    return new_user
```

The exact implementation can evolve, but the responsibility remains:

> Application/business logic for users.

---

# 20. Service vs Session

These are different.

### Service

```text
app/services/user_service.py
```

Responsible for:

```text
Business/application logic
```

### Session

```text
app/database/session.py
```

Responsible for:

```text
Creating/managing SQLAlchemy database sessions
```

Relationship:

```text
Service
   |
   | uses
   v
SQLAlchemy Session
   |
   | communicates with
   v
PostgreSQL
```

The service does not execute the session file.

It receives a session object and uses it.

---

# 21. `db.add()`

When the service calls:

```python
db.add(new_user)
```

SQLAlchemy begins tracking the new object in the current transaction.

It is not the same as permanently committing the row.

---

# 22. `db.commit()`

Then:

```python
db.commit()
```

commits the transaction.

Conceptually:

```text
new_user
   |
   v
db.add()
   |
   v
Current transaction
   |
   v
db.commit()
   |
   v
PostgreSQL
```

This is where the transaction is committed to PostgreSQL.

SQLAlchemy generates the appropriate SQL.

Conceptually, it is similar to:

```sql
INSERT INTO users (username, email)
VALUES ('hemanth', 'hemanth@gmail.com');
```

We normally do not have to write that SQL ourselves.

---

# 23. `db.refresh()`

After committing:

```python
db.refresh(new_user)
```

can refresh the Python object with values generated by the database.

For example:

```text
Before database insert:
id = ?

After commit/refresh:
id = 1
```

---

# 24. Swagger UI

FastAPI automatically provides Swagger UI, normally at:

```text
http://127.0.0.1:8000/docs
```

Swagger is currently our API testing client.

It is not our final frontend.

Later:

```text
React Frontend
      |
      v
FastAPI
```

Instead of:

```text
Swagger
      |
      v
FastAPI
```

Swagger is simply convenient during backend development.

---

# 25. Complete POST `/users` Flow

Suppose Swagger sends:

```http
POST /api/v1/users
Content-Type: application/json
```

Body:

```json
{
    "username": "hemanth",
    "email": "hemanth@gmail.com"
}
```

### Step 1 — Swagger

Swagger sends an HTTP request.

```text
Swagger UI
    |
    | POST /api/v1/users
    v
FastAPI
```

### Step 2 — Route

File:

```text
app/routes/users.py
```

FastAPI finds the `POST /users` route and calls it.

### Step 3 — Pydantic

The request body is mapped to:

```python
user: UserCreate
```

Pydantic validates it.

If invalid, FastAPI returns a validation error and the service is not called.

### Step 4 — Database dependency

FastAPI sees:

```python
Depends(get_db)
```

and calls:

```text
app/database/session.py
        |
        v
get_db()
        |
        v
SessionLocal()
        |
        v
SQLAlchemy Session
```

### Step 5 — Route calls service

```python
create_user(db, user)
```

The service receives:

```text
db
 |
 +--> SQLAlchemy Session

user
 |
 +--> Pydantic UserCreate
```

### Step 6 — Service creates SQLAlchemy object

```python
new_user = User(
    username=user.username,
    email=user.email,
)
```

This transfers values from the Pydantic object into a SQLAlchemy database model object.

```text
Pydantic UserCreate
       |
       | copy values
       v
SQLAlchemy User
```

### Step 7 — Add

```python
db.add(new_user)
```

### Step 8 — Commit

```python
db.commit()
```

SQLAlchemy communicates with PostgreSQL and commits the transaction.

### Step 9 — Database

PostgreSQL now stores the row in:

```text
users
```

For example:

```text
id | username | email
---+----------+--------------------
1  | hemanth  | hemanth@gmail.com
```

### Step 10 — Refresh

```python
db.refresh(new_user)
```

### Step 11 — Return

The service returns the result.

FastAPI serializes the response to JSON.

Swagger displays it.

### Step 12 — Close session

After the request finishes:

```python
db.close()
```

runs through the dependency cleanup.

---

# 26. Complete Runtime Flow — Memorize This

```text
Swagger UI / Frontend
        |
        v
POST /api/v1/users
        |
        v
FastAPI Route
        |
        v
Pydantic UserCreate
        |
        v
Validation
        |
        v
Depends(get_db)
        |
        v
get_db()
        |
        v
SessionLocal()
        |
        v
SQLAlchemy Session
        |
        v
user_service.py
        |
        v
Create SQLAlchemy User object
        |
        v
db.add()
        |
        v
db.commit()
        |
        v
PostgreSQL users table
        |
        v
db.refresh()
        |
        v
JSON response
        |
        v
Swagger UI
```

---

# 27. Database Structure Flow — Memorize Separately

When changing tables:

```text
app/models/user.py
        |
        v
User SQLAlchemy Model
        |
        v
Base.metadata
        |
        v
Alembic
        |
        v
Migration File
        |
        v
alembic upgrade head
        |
        v
PostgreSQL Schema
```

Examples:

```text
Create users table
Add password column
Add index
Create another table
Change a constraint
```

---

# 28. File-by-File Responsibility

| File | Responsibility |
|---|---|
| `.env` | Configuration values/secrets |
| `app/config/settings.py` | Loads and validates configuration |
| `app/main.py` | Creates FastAPI app and registers routers |
| `app/routes/health.py` | Health endpoint |
| `app/routes/users.py` | User HTTP endpoints |
| `app/schemas/user.py` | Pydantic request/response validation |
| `app/models/user.py` | SQLAlchemy database model |
| `app/database/base.py` | SQLAlchemy declarative Base |
| `app/database/session.py` | Engine, SessionLocal, `get_db()` |
| `app/services/user_service.py` | User business/application logic |
| `alembic/env.py` | Connects Alembic to config and SQLAlchemy metadata |
| `alembic/versions/` | Migration files |
| `alembic.ini` | Alembic configuration |

---

# 29. What We Have Built

## Application

- FastAPI application
- Uvicorn server
- Root endpoint
- Health endpoint
- User endpoint

## Configuration

- `.env`
- Pydantic Settings
- Centralized configuration

## Database

- PostgreSQL installed
- `devops_ai_platform` database created
- `users` table created
- `alembic_version` table created

## ORM

- SQLAlchemy
- SQLAlchemy Base
- User model
- Database engine
- Database session

## Migrations

- Alembic initialized
- Autogeneration configured
- User migration generated
- Migration applied

## Validation

- Pydantic `UserCreate`
- Username validation
- Email validation
- `EmailStr`

## API

- POST user endpoint
- Database dependency
- Service layer
- Swagger testing

---

# 30. What We Have Not Built Yet

Possible next stages:

```text
GET /users
GET /users/{id}
PUT /users/{id}
DELETE /users/{id}
```

Then:

```text
Authentication
JWT
Password hashing
Authorization
```

Eventually:

```text
AWS integration
Terraform integration
Kubernetes integration
CI/CD
Monitoring
AI features
DevOps tool integrations
React dashboard
```

---

# 31. Interview Explanation

If asked:

> Explain your project architecture.

A strong answer:

> "I'm building a DevOps AI Platform with FastAPI as the backend. I use Pydantic for request validation, SQLAlchemy as the ORM, PostgreSQL as the database, and Alembic for database schema migrations. The application follows a layered structure. Incoming requests enter through the route layer, Pydantic validates the request body, the route obtains a database session through FastAPI dependency injection, and then passes the validated data and session to the service layer. The service contains the application logic and uses SQLAlchemy to create or modify database records. SQLAlchemy communicates with PostgreSQL through the database session. Alembic is separate from the runtime request flow and is used to manage database schema changes."

---

# 32. What You Actually Need to Remember

Do not try to memorize every line.

Remember these relationships:

```text
.env
  ↓
settings.py
  ↓
Application configuration
```

```text
SQLAlchemy Model
  ↓
Alembic
  ↓
Database Table
```

```text
Pydantic Schema
  ↓
Validate API input
```

```text
Session
  ↓
Database communication
```

```text
Service
  ↓
Business logic
```

```text
Route
  ↓
HTTP/API layer
```

Most important runtime flow:

```text
Client
  ↓
FastAPI Route
  ↓
Pydantic
  ↓
get_db()
  ↓
Service
  ↓
SQLAlchemy Session
  ↓
PostgreSQL
```

---

# 33. Simple Mental Model

Think of a restaurant:

```text
Swagger / Frontend
       ↓
    Customer
       ↓
 FastAPI Route
       ↓
     Waiter
       ↓
    Pydantic
       ↓
Check that the order is valid
       ↓
    Service
       ↓
    Kitchen
       ↓
SQLAlchemy Session
       ↓
Database communication
       ↓
 PostgreSQL
       ↓
   Storage
```

Alembic is different:

```text
Alembic
   ↓
Changes the restaurant's structure
(tables/columns/indexes)
```

It is not normally involved in every customer order.

---

# 34. Final Mental Picture

```text
                         CLIENT
                           |
                    Swagger / Frontend
                           |
                           v
                    +-------------+
                    |   FastAPI   |
                    |    Route    |
                    +------+------+ 
                           |
                    Request Data
                           |
                           v
                    +-------------+
                    |   Pydantic  |
                    |  Validation |
                    +------+------+ 
                           |
                     Valid Data
                           |
                           v
                    +-------------+
                    |   Service   |
                    |    Logic    |
                    +------+------+ 
                           |
                    SQLAlchemy Model
                           |
                           v
                    +-------------+
                    |   Session   |
                    |  db.add()   |
                    |  db.commit()|
                    +------+------+ 
                           |
                           v
                    +-------------+
                    | PostgreSQL  |
                    |    users    |
                    +-------------+


DATABASE STRUCTURE FLOW

SQLAlchemy Models
       |
       v
    Alembic
       |
       v
   Migration
       |
       v
  PostgreSQL
```

**If you understand these two diagrams, you understand the architecture we have built so far.**
