# How to Run the Application

This guide covers everything you need to run, configure, and troubleshoot the **DockerTestApp** full-stack application.

---

## Architecture Overview

The application is structured into four main components:

| Component | Technology | Default Port | Description |
| :--- | :--- | :--- | :--- |
| **Frontend** | React (Create React App) | `3000` | User interface for managing users |
| **Backend** | Node.js + Express | `3001` | REST API connected to MongoDB |
| **Database** | MongoDB 4.4 | `27017` | Document database storing user data |
| **DB Admin UI** | Mongo Express | `8081` | Web-based MongoDB administration tool |

---

## Method 1: Running with Docker Compose (Recommended)

Docker Compose spins up the entire stack (Database, Mongo Express, Backend API, and Frontend) in isolated containers with a single command.

### 1. Prerequisites
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running.

### 2. Start all services
Run the following command from the root directory of the project:

```bash
docker compose up --build
```

> **Tip**: Run in detached (background) mode with:
> ```bash
> docker compose up --build -d
> ```

### 3. Access the Services

- **Frontend App**: [http://localhost:3000](http://localhost:3000)
- **Backend API**: [http://localhost:3001/users](http://localhost:3001/users)
- **Mongo Express UI**: [http://localhost:8081/mongo/](http://localhost:8081/mongo/)
  - **Username**: `sarvin`
  - **Password**: `sarvinpass`
- **MongoDB Direct**: `localhost:27017`
  - **Root Username**: `admin`
  - **Root Password**: `pass123`

### 4. Viewing Logs & Stopping Services

- **View live logs**:
  ```bash
  docker compose logs -f
  ```
- **Stop containers**:
  ```bash
  docker compose down
  ```
- **Stop containers and remove volumes (resets database)**:
  ```bash
  docker compose down -v
  ```

---

## Method 2: Running Locally (Development Mode)

If you prefer to run the Node.js services directly on your machine without containerizing the frontend/backend:

### Step 1: Start MongoDB
You still need a running MongoDB instance. You can launch just the database and Mongo Express using Docker:

```bash
docker compose up -d db mongoExpress
```

Alternatively, use the standalone `mongo.yaml`:
```bash
docker compose -f mongo.yaml up -d
```

### Step 2: Start Backend API

1. Open a new terminal and navigate to the `backend` folder:
   ```bash
   cd backend
   ```
2. Install dependencies (if not already installed):
   ```bash
   npm install
   ```
3. Start the server:
   ```bash
   npm start
   # or with auto-reload (nodemon):
   npm run dev
   ```
   The backend will be running at [http://localhost:3001](http://localhost:3001).

### Step 3: Start Frontend

1. Open a new terminal and navigate to the `frontend` folder:
   ```bash
   cd frontend
   ```
2. Install dependencies (if not already installed):
   ```bash
   npm install
   ```
3. Start the React development server:
   ```bash
   npm start
   # or:
   npm run dev
   ```
   > **Note on `npm run dev`**: Create React App natively uses `npm start`. A `dev` script alias has been added to `frontend/package.json` so both `npm start` and `npm run dev` work seamlessly.

4. Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## Environment Variables Configuration

| Variable | Location | Default / Example | Purpose |
| :--- | :--- | :--- | :--- |
| `DB_URL` | `backend` / `docker-compose.yaml` | `mongodb://admin:pass123@db:27017/?authSource=admin` | MongoDB connection string |
| `REACT_APP_API_URL` | `frontend` / `docker-compose.yaml` | `http://localhost:3001` | Base URL used by browser to query backend API |

> **Deploying to a Remote Server?**
> If running on a remote server (e.g., `188.40.180.113`), set `REACT_APP_API_URL` before launching Docker Compose:
> ```bash
> REACT_APP_API_URL=http://188.40.180.113:3001 docker compose up --build
> ```

---

## API Endpoints Reference

You can test the backend API using `curl` or Postman:

- **Get all users**:
  ```bash
  curl http://localhost:3001/users
  ```

- **Add a new user**:
  ```bash
  curl -X POST http://localhost:3001/users \
       -H "Content-Type: application/json" \
       -d '{"name": "Alice"}'
  ```

- **Delete a user**:
  ```bash
  curl -X DELETE http://localhost:3001/users/Alice
  ```

---

## Troubleshooting & FAQs

### 1. `npm error Missing script: "dev"`
- **Why it occurred**: Standard Create React App uses `npm start` rather than `npm run dev`.
- **Solution**: We added `"dev": "react-scripts start"` into `frontend/package.json` and `"dev": "nodemon index.js"` into `backend/package.json`. You can now run either `npm start` or `npm run dev`.

### 2. `MongoServerSelectionError: connect ECONNREFUSED`
- **Cause**: The backend started before MongoDB was ready or MongoDB is not running.
- **Solution**: If running locally, make sure MongoDB is up (`docker compose up -d db`). In Docker Compose, the backend container automatically restarts until MongoDB is available (`restart: always`).

### 3. Port Conflicts (Port 3000, 3001, 8081, or 27017 in use)
- Identify and terminate conflicting processes:
  ```bash
  # Check if port 3000 is occupied
  lsof -i :3000
  # Kill process by PID if needed
  kill -9 <PID>
  ```
