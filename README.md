# Task Manager API

A REST API built with Node.js and Express for managing tasks. Users can register, log in, and manage their own tasks. Built as part of a backend assignment.

---

## Tech Stack

- **Node.js + Express** — server and routing
- **PostgreSQL + Sequelize** — storing user data
- **MongoDB + Mongoose** — storing tasks
- **JWT** — authentication
- **bcrypt** — password hashing
- **express-validator** — input validation

---

## Folder Structure

```
src/
├── config/         # db connections (postgres + mongo)
├── controllers/    # request handling logic
├── middleware/     # auth + error handler
├── models/         # User (postgres), Task (mongo)
├── routes/         # route definitions
├── validators/     # input validation rules
└── server.js       # entry point
```

---

## Getting Started

### 1. Clone the repo

```bash
git clone <your-repo-url>
cd task-manager-api
```

### 2. Install dependencies

```bash
npm install
```

### 3. Set up environment variables

Copy `.env.example` to `.env` and fill in your values:

```bash
cp .env.example .env
```

```
PORT=5000
DB_HOST=localhost
DB_PORT=5432
DB_NAME=taskmanager
DB_USER=postgres
DB_PASSWORD=yourpassword
MONGO_URI=mongodb://localhost:27017/taskmanager
JWT_SECRET=pick_something_random_here
JWT_EXPIRES_IN=7d
```

### 4. Set up PostgreSQL

Make sure PostgreSQL is running and create the database:

```sql
CREATE DATABASE taskmanager;
```

Sequelize will auto-create the `users` table when the server starts (`sync({ alter: true })`).

### 5. Set up MongoDB

Just make sure MongoDB is running locally — the `taskmanager` database and `tasks` collection will be created automatically.

### 6. Run the server

```bash
# development (with nodemon)
npm run dev

# production
npm start
```

---

## API Endpoints

### Auth

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/api/auth/register` | Create new account | No |
| POST | `/api/auth/login` | Login, returns JWT | No |
| GET | `/api/auth/me` | Get current user info | Yes |

### Tasks

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/api/tasks` | Create a task | Yes |
| GET | `/api/tasks` | Get all your tasks | Yes |
| GET | `/api/tasks/:id` | Get one task by ID | Yes |
| PATCH | `/api/tasks/:id` | Update a task (partial) | Yes |
| DELETE | `/api/tasks/:id` | Delete a task | Yes |

For protected routes, send the token in the header:
```
Authorization: Bearer <your_token>
```

---

## Example Requests

### Register
```json
POST /api/auth/register
{
  "email": "test@example.com",
  "password": "secret123"
}
```

### Login
```json
POST /api/auth/login
{
  "email": "test@example.com",
  "password": "secret123"
}
```

### Create Task
```json
POST /api/tasks
Authorization: Bearer <token>

{
  "title": "Finish assignment",
  "description": "complete the backend task",
  "due_date": "2024-12-31",
  "status": "pending"
}
```

### Update Task (partial)
```json
PATCH /api/tasks/<id>
Authorization: Bearer <token>

{
  "status": "completed"
}
```

---

## Error Handling

All errors return JSON in this format:

```json
{
  "message": "some error message"
}
```

Validation errors return:
```json
{
  "errors": [
    { "msg": "title cant be empty", "path": "title", ... }
  ]
}
```

Common HTTP status codes used:
- `400` — bad request / validation error
- `401` — not logged in / bad token
- `403` — trying to access someone else's data
- `404` — resource not found
- `500` — server error

---

## Design Decisions

- **Why PostgreSQL for users and MongoDB for tasks?** Users are structured and relational, so SQL made sense. Tasks are more flexible and document-like, so Mongo fit better. The `userId` field in the Task schema stores the Postgres user ID to link them.
- **JWT stored client-side** — stateless auth, no sessions needed.
- **PATCH instead of PUT for updates** — allows partial updates so you don't need to resend the full task body.
- **Global error handler** — all async errors are passed to `next(err)` and handled in one place.

---

## Notes

- Make sure both PostgreSQL and MongoDB are running before starting the server
- Never commit your `.env` file — it's in `.gitignore`
- All sensitive config (JWT secret, DB credentials) goes in `.env` only
