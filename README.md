# 🚀 StaffPulse: Full-Stack Staff & Department Management System with Real-Time Container Monitoring

[![Node.js](https://img.shields.io/badge/Node.js-v18%2B-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![Express.js](https://img.shields.io/badge/Express.js-v4.19-000000?style=for-the-badge&logo=express&logoColor=white)](https://expressjs.com/)
[![MongoDB](https://img.shields.io/badge/MongoDB-v7.0-47A248?style=for-the-badge&logo=mongodb&logoColor=white)](https://www.mongodb.com/)
[![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Prometheus](https://img.shields.io/badge/Prometheus-Metrics-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-Dashboards-F46800?style=for-the-badge&logo=grafana&logoColor=white)](https://grafana.com/)

A modern, full-stack staff registry and department management platform built with **Node.js**, **Express**, **MongoDB**, and vanilla frontend technologies. It features **Role-Based Access Control (RBAC)** via JWT, relational schema safety, and containerized telemetry with **Prometheus** and **Grafana** tracking real-time application health and **CPU/Memory utilization**.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [Tech Stack](#-tech-stack)
- [Project Architecture](#-project-architecture)
- [Getting Started](#-getting-started)
  - [Option 1: Docker Compose (Recommended)](#option-1-docker-compose-recommended)
  - [Option 2: Local Development Setup](#option-2-local-development-setup)
- [API Reference](#-api-reference)
- [Real-Time Monitoring & Telemetry](#-real-time-monitoring--telemetry)
- [Environment Variables](#-environment-variables)
- [License](#-license)

---

## 💡 Overview

Managing staff and department hierarchies manually often results in data redundancy, broken foreign key associations, and unmonitored infrastructure. **StaffPulse** addresses these challenges by:
1. Enforcing relational data integrity between staff and department entities.
2. Securing user actions through stateless JWT authentication and role-based permissions.
3. Exposing Node.js process health (CPU, Memory, Uptime) to Prometheus and Grafana for continuous monitoring.

> [!NOTE]
> Deleting a department that still has staff members assigned to it is automatically blocked by the backend to prevent orphaned database records.

---

## ✨ Key Features

- **🔐 Dual Access Roles (User vs. Admin)**
  - **User:** Account creation, secure authentication, and new staff registration.
  - **Admin:** Directory management, searching/filtering, editing/deleting staff, and full department control.
- **🛡️ Security & Authentication**
  - Passwords hashed using `bcryptjs` before storage in MongoDB.
  - Stateless authentication via **JSON Web Tokens (JWT)** passed in `Authorization: Bearer <token>` headers.
  - Backend authorization guards (`requireAdmin`) protecting sensitive endpoints.
- **🔗 Relational Schema Integrity**
  - Mongoose `ObjectId` references connecting staff to departments.
  - Dynamic population of department metadata and real-time staff count aggregations.
  - Safe-delete protection on department documents.
- **📊 Real-Time Server Telemetry (CPU & Memory)**
  - Integrated `prom-client` collecting process metrics (CPU usage, memory heap/RSS, event loop lag, HTTP requests).
  - Out-of-the-box Prometheus scraping and pre-configured Grafana visualization.

---

## 🛠️ Tech Stack

| Domain | Technologies |
| :--- | :--- |
| **Backend** | Node.js, Express.js |
| **Database** | MongoDB, Mongoose ODM |
| **Authentication** | JSON Web Tokens (JWT), `bcryptjs` |
| **Frontend** | HTML5, CSS3 (Fraunces & Inter fonts), Vanilla JavaScript (ES6+) |
| **DevOps & Monitoring** | Docker, Docker Compose, Prometheus, Grafana |

---

## 📁 Project Architecture

```
staff-registration/
├── config/
│   └── db.js                 # MongoDB connection handler
├── middleware/
│   └── auth.js               # JWT authentication & admin authorization guards
├── models/
│   ├── User.js               # User schema with bcrypt password pre-save hook
│   ├── Staff.js              # Staff schema with department ObjectId reference
│   └── Department.js         # Department schema
├── routes/
│   ├── auth.js               # User signup, login, and admin authentication
│   ├── staff.js              # Staff CRUD & regex search endpoints
│   └── department.js         # Department CRUD & live staff count aggregation
├── public/
│   ├── login.html            # User & Admin login interface
│   ├── signup.html           # User account creation interface
│   ├── index.html            # Staff registration form
│   ├── staff.html            # Staff directory & search dashboard (Admin)
│   └── departments.html      # Department management grid (Admin)
├── prometheus/
│   └── prometheus.yml        # Prometheus target & scrape configuration
├── Dockerfile                # Multi-stage Docker container build definition
├── docker-compose.yml        # Orchestration for Node app, Mongo, Prometheus & Grafana
└── server.js                 # Express application entrypoint & /metrics exporter
```

---

## 🚀 Getting Started

### Option 1: Docker Compose 

Run the complete multi-container setup (Node.js App, MongoDB, Prometheus, Grafana) with a single command:

```bash

#  Build and launch all services
docker-compose up --build
```

#### 🌐 Service Endpoints:
- **Web Application:** `http://localhost:5000`
- **Prometheus Server:** `http://localhost:9090`
- **Grafana Dashboard:** `http://localhost:3050` *(Credentials: `admin` / `admin`)*

---

### Option 2: Local Development Setup

#### Prerequisites
- [Node.js (v18+)](https://nodejs.org/)
- [MongoDB](https://www.mongodb.com/) running locally on port `27017`

```bash
# 1. Install dependencies
npm install

# 2. Create environment configuration
cp .env.example .env

# 3. Start the application in development mode
npm run dev
```

> [!TIP]
> Ensure MongoDB is active before launching the server locally using `mongod`.

---

## 📡 API Reference

### Auth Endpoints (`/api/auth`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/auth/signup` | Public | Register a new regular user account |
| `POST` | `/api/auth/login` | Public | Authenticate a user and return a JWT |
| `POST` | `/api/auth/admin-login` | Public | Authenticate as Admin using `.env` credentials |
| `GET` | `/api/auth/me` | Authenticated | Retrieve current user identity |

### Staff Endpoints (`/api/staff`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/staff` | Authenticated | Register a new staff record |
| `GET` | `/api/staff` | Admin Only | Fetch all staff (supports `?search=` and `?department=`) |
| `GET` | `/api/staff/:id` | Admin Only | Fetch detailed record for a specific staff member |
| `PUT` | `/api/staff/:id` | Admin Only | Update an existing staff record |
| `DELETE` | `/api/staff/:id` | Admin Only | Remove a staff record |

### Department Endpoints (`/api/departments`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/departments` | Admin Only | Create a new department |
| `GET` | `/api/departments` | Authenticated | List departments with live `staffCount` |
| `GET` | `/api/departments/:id` | Admin Only | Fetch single department details |
| `PUT` | `/api/departments/:id` | Admin Only | Update department details |
| `DELETE` | `/api/departments/:id` | Admin Only | Delete department (blocked if staff assigned) |

---

## 📊 Real-Time Monitoring & Telemetry

The server integrates `prom-client` to measure system health and performance:

```
[ Express Server ] ──(/metrics)──> [ Prometheus ] ──(Visualization)──> [ Grafana ]
```

### Metrics Tracked:
- **Process CPU Usage:** User CPU time, system CPU time, and total CPU utilization percentage.
- **Memory Consumption:** Process Resident Set Size (RSS), Heap Total, and Heap Used.
- **Node.js Runtime:** Event loop lag, active handles, process uptime, and garbage collection metrics.

---

## ⚙️ Environment Variables

Configure  `.env` file with the following keys:

```env
PORT=5000
MONGO_URI=mongodb://127.0.0.1:27017/staffDB
JWT_SECRET=my_super_secret_jwt_key_change_in_production
JWT_EXPIRES_IN=7d

# Fixed Admin Credentials
ADMIN_USERNAME=admin
ADMIN_PASSWORD=admin
```

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.
