# 🏢 HRMS SaaS — Human Resource Management System

> A full-stack **Human Resource Management System** built with **Django 6**, **Django REST Framework**, and a **Vanilla JS SPA** frontend. Designed for role-based access, JWT authentication, and dual-database support (MySQL locally, SQLite on cloud/Vercel).

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Data Models](#data-models)
- [API Endpoints](#api-endpoints)
- [Role-Based Access Control](#role-based-access-control)
- [Frontend Architecture](#frontend-architecture)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Local Setup (MySQL via XAMPP)](#local-setup-mysql-via-xampp)
  - [Running the Development Server](#running-the-development-server)
  - [Creating a Superuser (Admin)](#creating-a-superuser-admin)
- [Environment & Configuration](#environment--configuration)
- [Database Configuration](#database-configuration)
- [Deployment (Vercel)](#deployment-vercel)
- [Screenshots / UI Overview](#screenshots--ui-overview)
- [License](#license)

---

## Overview

**HRMS SaaS** is a single-page application (SPA) served by a Django backend. The entire UI runs in the browser using vanilla JavaScript, fetching data from a RESTful API protected by **JWT (JSON Web Tokens)**. The application supports two distinct user roles — **Admin (HR Manager)** and **Employee** — each with a tailored interface and enforced server-side permissions.

---

## Features

### 🔐 Authentication
- JWT-based login with access and refresh token lifecycle (1-day access, 7-day refresh)
- Tokens stored in `localStorage`; automatic logout on token expiry (401 response)
- Dynamic avatar and profile name rendered in the topbar after login

### 📊 Dashboard
- Animated stat counters for: **Total Employees**, **Present Today**, **On Leave**, **Total Payroll**
- Admins see org-wide aggregates; Employees see only their own metrics

### 👥 Employee Management *(Admin only)*
- Add new employees (Name, Email, Department, Salary)
- View all employees in a styled data table
- Delete employees

### 🗓️ Attendance Tracking
- **Admin**: Mark attendance (Present / Absent) for all employees on any selected date; bulk save
- **Employee**: View personal attendance history sorted by most recent; one-click "Mark Present Today" check-in

### 🌴 Leave Management
- Employees can submit leave requests (Employee, Reason, Start Date, End Date)
- Admins can Approve or Reject pending leave requests
- Employees can cancel their own pending requests
- Leave status displayed with colour-coded badges: Pending (amber), Approved (green), Rejected (red)

### 💰 Payroll
- **Admin**: Add payroll records (Employee, Month, Total Salary); view all salary distributions; delete records; see total cycle distribution
- **Employee**: Read-only salary slip view ("My Salary Slips") with a "✓ Paid" badge per entry and total earnings summary

### 🖥️ UI / UX
- Collapsible sidebar with Lucide icons
- Glassmorphism login card with fade-up animation
- Animated number counters on dashboard
- Toast notification system for user feedback
- Animated modal dialogs with floating label inputs
- Fully responsive table layout with horizontal scroll

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Backend Framework** | Django 6.0.3 |
| **REST API** | Django REST Framework |
| **Authentication** | `djangorestframework-simplejwt` |
| **CORS** | `django-cors-headers` |
| **Static Files (prod)** | WhiteNoise |
| **Database (local)** | MySQL 8 via XAMPP (PyMySQL adapter) |
| **Database (cloud)** | SQLite (Vercel / ephemeral) |
| **Frontend** | Vanilla JavaScript (ES2020+) |
| **Styling** | Vanilla CSS with CSS custom properties |
| **Icons** | [Lucide Icons](https://lucide.dev/) (CDN) |
| **Fonts** | Google Fonts — Inter |

---

## Project Structure

```
HRMS_S2/
├── hrms_project/               # Django project package
│   ├── __init__.py             # PyMySQL adapter + MariaDB monkeypatches
│   ├── settings.py             # All project settings (DB, JWT, CORS, etc.)
│   ├── urls.py                 # Root URL configuration
│   ├── wsgi.py                 # WSGI entry point
│   └── asgi.py                 # ASGI entry point
│
├── employees/                  # Core Django app
│   ├── models.py               # Employee, Attendance, Leave, Payroll models
│   ├── serializers.py          # DRF serializers for all models
│   ├── views.py                # ViewSets + custom APIViews
│   ├── urls.py                 # App-level URL routing + JWT endpoints
│   ├── admin.py                # Django admin registrations
│   ├── forms.py                # ModelForms (Employee, Attendance, Leave)
│   └── migrations/
│       ├── 0001_initial.py     # Initial schema creation
│       └── 0002_employee_user.py # Added OneToOne link to Django User
│
├── templates/
│   └── index.html              # SPA shell: Login view + Dashboard layout + Modals
│
├── static/
│   ├── css/
│   │   └── style.css           # Full design system (variables, layout, components)
│   └── js/
│       └── app.js              # All frontend logic (auth, routing, CRUD, UI)
│
├── manage.py                   # Django management CLI
└── db.sqlite3                  # Fallback local SQLite database
```

---

## Data Models

### `Employee`
| Field | Type | Notes |
|---|---|---|
| `id` | BigAutoField | Primary key |
| `user` | OneToOneField → User | Optional link to Django auth user |
| `name` | CharField(100) | Full name |
| `email` | EmailField | Contact email |
| `department` | CharField(100) | e.g. Engineering, HR |
| `salary` | FloatField | Base salary amount |

### `Attendance`
| Field | Type | Notes |
|---|---|---|
| `id` | BigAutoField | Primary key |
| `employee` | ForeignKey → Employee | CASCADE delete |
| `date` | DateField | Attendance date |
| `status` | CharField(10) | `"Present"` or `"Absent"` |

### `Leave`
| Field | Type | Notes |
|---|---|---|
| `id` | BigAutoField | Primary key |
| `employee` | ForeignKey → Employee | CASCADE delete |
| `reason` | TextField | Leave justification |
| `start_date` | DateField | Leave start |
| `end_date` | DateField | Leave end |
| `status` | CharField(20) | `"Pending"`, `"Approved"`, `"Rejected"` |

### `Payroll`
| Field | Type | Notes |
|---|---|---|
| `id` | BigAutoField | Primary key |
| `employee` | ForeignKey → Employee | CASCADE delete |
| `month` | CharField(20) | e.g. `"2024-03"` |
| `total_salary` | FloatField | Total salary disbursed |

---

## API Endpoints

All API endpoints are prefixed with `/api/`.

### Authentication

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/token/` | Obtain JWT access + refresh tokens |
| `POST` | `/api/token/refresh/` | Refresh an expired access token |

### User Info

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/me/` | Returns current user's role, username, and linked employee ID |

### Dashboard

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/dashboard/summary/` | Returns aggregated HR metrics |

### Employees

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/employees/` | List all employees |
| `POST` | `/api/employees/` | Create a new employee |
| `GET` | `/api/employees/{id}/` | Retrieve a single employee |
| `PUT/PATCH` | `/api/employees/{id}/` | Update an employee |
| `DELETE` | `/api/employees/{id}/` | Delete an employee |

### Attendance

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/attendance/` | Admin: all records; Employee: own records |
| `POST` | `/api/attendance/` | Create an attendance record |
| `DELETE` | `/api/attendance/{id}/` | Delete an attendance record |

### Leave

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/leave/` | Admin: all requests; Employee: own requests |
| `POST` | `/api/leave/` | Submit a leave request |
| `PATCH` | `/api/leave/{id}/` | Admin: update status (Approve/Reject) |
| `DELETE` | `/api/leave/{id}/` | Delete a leave request |

### Payroll

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/payroll/` | Admin: all records; Employee: own salary slips |
| `POST` | `/api/payroll/` | Admin only: create a payroll record |
| `PUT/PATCH` | `/api/payroll/{id}/` | Admin only: update a payroll record |
| `DELETE` | `/api/payroll/{id}/` | Admin only: delete a payroll record |

> All endpoints except `/api/token/` and `/api/token/refresh/` require a `Bearer <access_token>` Authorization header.

---

## Role-Based Access Control

The system enforces two roles from Django's built-in auth system:

| Capability | Admin (`is_staff` or `is_superuser`) | Employee |
|---|:---:|:---:|
| View all employees | ✅ | ❌ (nav hidden) |
| Add / delete employees | ✅ | ❌ |
| Mark attendance for all | ✅ | ❌ |
| Self check-in | ❌ | ✅ |
| View own attendance history | — | ✅ |
| Submit leave requests | ✅ | ✅ |
| Approve / Reject leave | ✅ | ❌ |
| Cancel own pending leave | ✅ | ✅ |
| Add payroll records | ✅ | ❌ |
| View all payroll | ✅ | ❌ |
| View own salary slips | — | ✅ |
| Delete payroll records | ✅ | ❌ |
| Dashboard — org-wide metrics | ✅ | ❌ |
| Dashboard — personal metrics | — | ✅ |

Server-side enforcement is handled in each ViewSet's `get_queryset`, `create`, `update`, `partial_update`, and `destroy` methods. Attempting a restricted operation returns a `403 PermissionDenied` response.

---

## Frontend Architecture

The frontend is a **single-page application (SPA)** rendered entirely in the browser, served from `templates/index.html` by the Django `app_view` catch-all route.

### `static/js/app.js` (686 lines)

| Responsibility | Key Functions |
|---|---|
| Auth state & routing | `showApp()`, `showLogin()`, `fetchUserRole()` |
| Role-based UI | `applyRoleUI()` — hides/shows nav items and action buttons |
| Dashboard | `loadDashboardData()`, `animateValue()` — animated counters |
| Employees CRUD | `loadEmployees()`, `deleteEmployee()` |
| Attendance | `loadAttendanceSection()`, `loadAttendanceAdmin()`, `loadAttendanceEmployee()` |
| Leave CRUD | `loadLeave()`, `updateLeaveStatus()`, `deleteLeave()` |
| Payroll CRUD | `loadPayroll()`, `deletePayroll()` |
| API wrapper | `apiFetch()` — injects JWT header, handles 401 auto-logout |
| Modals | `openModal()`, `.close-modal` event listeners |
| Notifications | `showToast()` — auto-dismissing bottom-right toast |

### `static/css/style.css` (476 lines)

Built on a **CSS custom properties design system**:

```css
--primary: #0F3D2E;        /* Deep green — brand color */
--primary-light: #1F7A63;  /* Hover / accent */
--success: #10B981;         /* Green badges */
--warning: #F59E0B;         /* Amber badges */
--danger: #EF4444;          /* Red / destructive */
```

Key component patterns: glassmorphism auth card, collapsible sidebar, stat cards with hover lift, floating label inputs, animated modal backdrops, slide-in toast notifications.

---

## Getting Started

### Prerequisites

- Python 3.10+
- pip
- XAMPP (for MySQL locally) **or** any MySQL 8 server
- A virtual environment tool (`venv` or `virtualenv`)
- Git (optional)

### Local Setup (MySQL via XAMPP)

1. **Clone or download the project**
   ```bash
   git clone <your-repo-url>
   cd HRMS_S2
   ```

2. **Create and activate a virtual environment**
   ```bash
   python -m venv venv
   # Windows
   venv\Scripts\activate
   # macOS / Linux
   source venv/bin/activate
   ```

3. **Install dependencies**

   > ⚠️ No `requirements.txt` is bundled. Install the following packages:

   ```bash
   pip install django==6.0.3
   pip install djangorestframework
   pip install djangorestframework-simplejwt
   pip install django-cors-headers
   pip install whitenoise
   pip install pymysql
   ```

4. **Start XAMPP** and ensure **MySQL** is running on port `3306`.

5. **Create the database** in phpMyAdmin or via MySQL CLI:
   ```sql
   CREATE DATABASE employee_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```

6. **Run migrations**
   ```bash
   python manage.py migrate
   ```

7. **Collect static files** (required for WhiteNoise to serve them)
   ```bash
   python manage.py collectstatic --no-input
   ```

### Running the Development Server

```bash
python manage.py runserver
```

Open your browser at **http://127.0.0.1:8000/**

### Creating a Superuser (Admin)

```bash
python manage.py createsuperuser
```

Follow the prompts to set a username and password. Log in to the app at `http://127.0.0.1:8000/` using these credentials to access the full Admin interface.

> **Linking an Employee profile to a User:**  
> Go to the Django Admin panel at `http://127.0.0.1:8000/admin/`, open an `Employee` record, and set its `user` field to a Django User. This enables the employee's self-service features (self check-in, personal salary slips, etc.).

---

## Environment & Configuration

Key settings in `hrms_project/settings.py`:

| Setting | Default | Description |
|---|---|---|
| `SECRET_KEY` | `'django-insecure-...'` | **Change this** before any production deployment |
| `DEBUG` | `True` (from env) | Set `DEBUG=False` in production |
| `ALLOWED_HOSTS` | `['*']` | Restrict to your domain in production |
| `CORS_ALLOW_ALL_ORIGINS` | `True` | Restrict to specific origins in production |
| `ACCESS_TOKEN_LIFETIME` | 1 day | JWT access token validity |
| `REFRESH_TOKEN_LIFETIME` | 7 days | JWT refresh token validity |
| `AUTH_HEADER_TYPES` | `Bearer` | Token prefix in `Authorization` header |

---

## Database Configuration

The project auto-selects the database based on the environment:

```python
if os.environ.get('VERCEL'):
    # SQLite — for serverless/ephemeral cloud environments
    DATABASES = { 'default': { 'ENGINE': '...sqlite3', 'NAME': '/tmp/db.sqlite3' } }
else:
    # MySQL — for local development with XAMPP
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'employee_db',
            'USER': 'root',
            'PASSWORD': '',
            'HOST': 'localhost',
            'PORT': '3306',
        }
    }
```

The `hrms_project/__init__.py` file installs **PyMySQL** as the MySQL adapter and patches Django 6 + MariaDB 10.4 compatibility issues:
- Disables the database version support check
- Disables `RETURNING` syntax (not supported on MariaDB 10.4)

---

## Deployment (Vercel)

To deploy on Vercel:

1. Add a `vercel.json` and `requirements.txt` to the project root.
2. Set the `VERCEL=1` environment variable in Vercel's dashboard — this switches the database to `/tmp/db.sqlite3` automatically.
3. Set `SECRET_KEY`, `DEBUG=False`, and `ALLOWED_HOSTS` as Vercel environment variables.
4. Run `python manage.py collectstatic` as part of your build step so WhiteNoise can serve static assets.

> **Note:** SQLite on Vercel's ephemeral filesystem means data does **not** persist across deployments or function cold starts. For production use, connect to a persistent database such as PlanetScale, Railway MySQL, or Supabase PostgreSQL.

---

## Screenshots / UI Overview

| Screen | Description |
|---|---|
| **Login** | Glassmorphism card with Inter font, brand green palette, `fadeUp` entry animation |
| **Dashboard** | Animated stat counters — Employees, Present Today, On Leave, Total Payroll |
| **Employees** | Sortable table with department badges and delete actions (Admin only) |
| **Attendance** | Admin bulk-mark table with radio toggles; Employee history with colour-coded status chips |
| **Leave** | Full leave management table; Admin Approve/Reject controls; Employee cancel option |
| **Payroll** | Admin full payroll table with cycle total; Employee personal salary slip view with "✓ Paid" badge |

---

## License

This project is licensed under the **MIT License** — see the [LICENSE](./LICENSE) file for details.