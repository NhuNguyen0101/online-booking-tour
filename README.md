# Online Booking Tour

A full-stack web application for browsing, booking and managing travel tours (the sample data is Vietnamese destinations such as Ha Long, Hoi An, Phu Quoc, Da Lat and Nha Trang).

- **Frontend:** React 18 + Vite + Tailwind CSS + Material UI / Material Tailwind
- **Backend:** Node.js + Express REST API (ES modules)
- **Database:** MySQL 8 (schema + sample data in `booking_tour.sql`)

> **Project status:** work in progress. The backend API and database are fairly complete. The frontend login/register pages talk to this backend, but several other pages (home slider, popular tours, cart, create tour) still contain code inherited from a course-selling template and call a third-party demo API. See [Known issues](#9-known-issues--limitations) before you start.

---

## Table of contents

1. [Features](#1-features)
2. [Tech stack](#2-tech-stack)
3. [What to install first (prerequisites)](#3-what-to-install-first-prerequisites)
4. [Step-by-step setup](#4-step-by-step-setup)
5. [Running the project](#5-running-the-project)
6. [Project structure](#6-project-structure)
7. [Database](#7-database)
8. [REST API reference](#8-rest-api-reference)
9. [Known issues / limitations](#9-known-issues--limitations)
10. [Other npm scripts](#10-other-npm-scripts)
11. [Troubleshooting](#11-troubleshooting)
12. [Deployment](#12-deployment)

---

## 1. Features

**Implemented in the backend (REST API)**

- Register / login with bcrypt-hashed passwords
- Four user roles: `administrator`, `client`, `tour_operator`, `tour_guide`
- Tours: list, search (by name, price range, sort by name/price/rating), create, update, delete
- Tour details: activities, images, locations, accommodations, vehicles
- Reviews and ratings per tour (average rating is computed in the tour list)
- Bookings (created as `Pending`, status can be updated)
- Payments (create, list, update status)
- Booking cancellations / refunds
- User contact info and role-specific profile data

**Implemented in the frontend**

- Home page with slider, sections and popular tours carousel
- Login and registration pages (Formik + Yup validation) connected to the backend
- Tour list / search page, tour detail page, cart page
- Admin-style "create tour" pages with a sidebar dashboard layout
- Responsive layout (Tailwind CSS)

---

## 2. Tech stack

| Area | Technology |
|---|---|
| UI library | [React 18](https://react.dev/) |
| Build tool | [Vite 5](https://vitejs.dev/) |
| Routing | react-router-dom 6 |
| State | Redux Toolkit (global), `useState` (local) |
| HTTP client | axios |
| Styling | Tailwind CSS 3, Material UI (`@mui/material`, `@mui/joy`), Material Tailwind, Heroicons, react-icons |
| Forms / validation | Formik + Yup |
| Alerts / toasts | SweetAlert2, react-toastify |
| Sliders | Swiper |
| Charts | Chart.js, `@mui/x-charts` |
| Auth helper | react-auth-kit (cookie), Google OAuth (`@react-oauth/google`, optional) |
| Image upload | Cloudinary (optional) |
| Backend | Node.js, Express, `mysql2`, `bcrypt`, `dotenv`, `cors` |
| Database | MySQL 8 |
| Testing / linting | Jest, Testing Library, ESLint |
| Hosting | Vercel (frontend, optional) |

---

## 3. What to install first (prerequisites)

You need these programs on your computer **before** running the project:

| Tool | Version | Why | Download |
|---|---|---|---|
| **Node.js** (includes `npm`) | 18 or newer (20 LTS recommended) | Runs the backend and the frontend dev server | https://nodejs.org |
| **MySQL Server** | 8.0 or newer (the dump was created with 8.4; MariaDB is **not** recommended) | Stores the data | https://dev.mysql.com/downloads/mysql/ |
| **A MySQL client** (pick one): MySQL Workbench, phpMyAdmin (via XAMPP), or the `mysql` command line | any | To import `booking_tour.sql` | https://dev.mysql.com/downloads/workbench/ |
| **Git** | any | To clone / push the repository | https://git-scm.com |
| A code editor (optional) | VS Code | Editing files | https://code.visualstudio.com |

Check your installation by opening a terminal (PowerShell / Command Prompt on Windows, Terminal on macOS/Linux):

```bash
node -v      # should print v18.x or higher
npm -v
mysql --version
git --version
```

> **Windows tip:** if `mysql` is "not recognized", either use MySQL Workbench / phpMyAdmin instead, or add the MySQL `bin` folder (for example `C:\Program Files\MySQL\MySQL Server 8.4\bin`) to your PATH.

Optional accounts (only for extra features):

- A free [Cloudinary](https://cloudinary.com) account – for uploading tour images
- A Google Cloud OAuth client – only if you want "Sign in with Google"

---

## 4. Step-by-step setup

### Step 1 – Get the code

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

(or just open a terminal inside the project folder if you already have it).

### Step 2 – Install the npm dependencies

Run this **in the project root folder** (the one containing `package.json`). One `package.json` serves both the frontend and the backend.

```bash
npm install
```

The backend also needs the `express` and `cors` packages. They are imported by `index.js` but are **not listed** in `package.json`, so install them explicitly:

```bash
npm install express cors
```

(If you are the maintainer, run this once and commit the updated `package.json` / `package-lock.json` so newcomers do not need this extra step.)

### Step 3 – Create the database

1. Start MySQL (on Windows: Services → *MySQL80* → Start, or start MySQL in XAMPP).
2. Create an empty database called `booking_tour`. The SQL file does **not** create it for you.
3. Import `booking_tour.sql` into it.

**Option A – command line**

```bash
mysql -u root -p -e "CREATE DATABASE booking_tour CHARACTER SET utf8mb4;"
mysql -u root -p booking_tour < booking_tour.sql
```

(`-p` makes MySQL ask for your password. On Windows PowerShell, `<` redirection does not work; use Command Prompt (`cmd`) for the second command, or use Option B/C.)

**Option B – MySQL Workbench**

1. Schema panel → right-click → *Create Schema…* → name it `booking_tour` → Apply.
2. *Server → Data Import* → *Import from Self-Contained File* → choose `booking_tour.sql` → Default Target Schema = `booking_tour` → *Start Import*.

**Option C – phpMyAdmin (XAMPP)**

1. Open http://localhost/phpmyadmin → *New* → database name `booking_tour`, collation `utf8mb4_0900_ai_ci` (or `utf8mb4_general_ci`) → *Create*.
2. Select the database → *Import* tab → choose `booking_tour.sql` → *Go*.

The file creates 20 tables, 4 stored procedures, a trigger and sample data (see [Database](#7-database)).

### Step 4 – Configure environment variables

Create a file named **`.env`** in the project root (same folder as `package.json`). It is git-ignored, so your passwords are never uploaded to GitHub.

```env
# ---------- Backend (read by index.js and backend/config/database.js) ----------
PORT=3000
CORS_ORIGIN=http://localhost:5173

MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_USER=root
MYSQL_PASSWORD=your_mysql_password
MYSQL_DB=booking_tour

# ---------- Frontend (optional – only for Cloudinary image upload) ----------
VITE_REACT_APP_CLOUD_NAME=your_cloudinary_cloud_name
VITE_REACT_APP_APIKEY=your_cloudinary_api_key
VITE_REACT_APP_APISECRET=your_cloudinary_api_secret
```

Notes:

- `MYSQL_HOST/USER/PASSWORD/DB/PORT` are **required**; the backend cannot connect without them.
- `PORT` defaults to `3000`. The frontend currently has `http://localhost:3000/` hard-coded in `src/pages/Login.jsx` and `src/pages/Register.jsx`, so keep `3000` unless you change those files.
- `CORS_ORIGIN` is the URL of the frontend (Vite's default is `http://localhost:5173`). If you leave it out, the API accepts requests from any origin (`*`), which is fine for local development only.
- Only variables starting with `VITE_` are visible to the frontend. Never put real secrets in a `VITE_` variable for a production site.
- Do not commit `.env`. (`.gitignore` already excludes `*.env`.)

### Step 5 – (Important) Make the sample accounts usable

The sample users in `booking_tour.sql` have **plain-text** passwords (for example `123456`), but the login code compares against **bcrypt hashes**, so the sample accounts **cannot log in as-is**. Choose one:

- **Easiest:** open the app and create a new account on the *Register* page (it stores a properly hashed password).
- Or replace a sample password with a bcrypt hash. Generate one:

  ```bash
  node -e "import('bcrypt').then(b => b.default.hash('123456', 10).then(console.log))"
  ```

  Then run in MySQL (use the hash that was printed):

  ```sql
  UPDATE users SET user_password = '<paste_hash_here>' WHERE user_name = 'client1@example.com';
  ```

Sample usernames (they are e-mail addresses): `admin1@example.com` … `admin5@example.com`, `client1@example.com` … `client10@example.com`, `tour_operator1@example.com` … `tour_operator5@example.com`, `tourguide01@example.com` … `tourguide05@example.com`.

---

## 5. Running the project

You need **two terminals** open at the same time, both in the project root.

**Terminal 1 – backend API (port 3000)**

```bash
node index.js
```

Expected output:

```
Server running on port 3000
Connected to MySQL successfully!
```

If you see `Error connecting to MySQL: ...`, re-check the `.env` values and that MySQL is running (the backend retries every 2 seconds).

**Terminal 2 – frontend (Vite dev server)**

```bash
npm run dev
```

Open the URL it prints, normally **http://localhost:5173**.

Quick test that the API works – open http://localhost:3000/api/tours in a browser; you should see JSON with the sample tours.

Useful pages in the frontend:

| URL | Page |
|---|---|
| `/` | Home |
| `/auth/register` | Register |
| `/auth/login` | Login |
| `/popular_tours` | Popular tours list |
| `/popular_tours/search` | Tour search |
| `/course/:courseId` | Tour detail |
| `/cart/` | Cart |
| `/profile/` | User profile |
| `/create/tour`, `/create/new_tour`, `/create/video` | Create tour (dashboard layout) |

After a successful login the frontend stores `userId`, `role` and `username` in the browser's `localStorage`.

---

## 6. Project structure

```
.
├── index.js                  # Backend entry point (Express server, mounts the routes)
├── backend/
│   ├── config/database.js    # MySQL connection (auto-reconnect), reads .env
│   ├── routes/               # URL → controller mapping
│   │   ├── auth.route.js         # /api/auth
│   │   ├── tour.route.js         # /api/tours
│   │   ├── review.route.js       # /api/reviews
│   │   └── user.route.js         # /api/users (profile, bookings, cancellations, payments)
│   ├── controllers/          # Request handling / validation / HTTP responses
│   │   └── auth, tour, review, user, booking, payment, cancellation controllers
│   └── models/               # SQL queries, one file per table (user, tour, booking, payment, ...)
├── booking_tour.sql          # Database schema + stored procedures + trigger + sample data
├── index.html                # Vite HTML entry (loads src/main.jsx, Poppins font)
├── src/                      # Frontend (React)
│   ├── main.jsx                  # App bootstrap (Redux Provider, auth provider, theme)
│   ├── apps/                     # App.jsx (all routes) and store.js (Redux store)
│   ├── pages/                    # Route-level pages (Home, Login, Register, Cart, CreateTour, ...)
│   ├── layouts/                  # Shared page layouts (Layout, AuthLayout, AdminDashboardLayout, ...)
│   ├── components/               # Reusable components (+ sub-folders per feature:
│   │                             #   Home_Component, OurCourse, create_tour, admin_dashboard)
│   ├── apis/                     # axios / RTK Query helper functions
│   ├── configs/                  # API base URL, SweetAlert config, global CSS
│   ├── constants/                # Hard-coded UI data (sidebar items, tabs, API base URL, mock data)
│   ├── features/slices/, slices/ # Redux slices
│   ├── hooks/                    # Custom hooks (useFetch, useLocalStorage, ...)
│   ├── utils/                    # Helper functions
│   ├── assets/                   # Images and logos
│   └── test/__mock__/            # Jest mocks for images and CSS
├── public/                   # Static files served as-is
├── vite.config.js            # Vite config (React plugin, Vercel plugin, "@" alias → /src)
├── tailwind.config.js, postcss.config.js
├── jest.config.cjs, babel.config.cjs     # Unit-test config
├── .eslintrc.cjs             # Lint rules
├── vercel.json               # SPA rewrites for Vercel
└── package.json
```

**Backend request flow:** `route → controller → model → MySQL`. To add a feature, add a SQL method in `backend/models`, call it from a controller, and expose it in a route file.

**Import alias:** in the frontend, `@/` means `src/` (for example `import Home from "@/pages/Home"`).

---

## 7. Database

Database name: **`booking_tour`** (MySQL 8, `utf8mb4`).

| Group | Tables |
|---|---|
| Users | `users`, `contact_detail`, `administrator`, `client`, `tour_operator`, `tour_guide` |
| Tours | `tour`, `tour_activities`, `tour_images`, `locations`, `accommodation`, `vehicle` |
| Tour links (many-to-many) | `tour_location`, `tour_accommodation`, `tour_vehicle` |
| Sales | `booking`, `payment`, `cancellation`, `reviews`, `manage` (admin ↔ booking) |

`users.user_role` is one of `administrator`, `client`, `tour_operator`, `tour_guide`. Each role has an extra table keyed by `user_id` that holds role-specific data (admin `level`, client `client_type`, operator `to_type`, guide `description`).

**Stored procedures**

| Name | Purpose |
|---|---|
| `insert_user` | Creates a row in `users` and in the matching role table in one transaction. Used by `POST /api/auth/register`. |
| `cancel_booking` | Sets a booking to `Cancelled` and adds a `cancellation` row. |
| `UpdateTourDates` | Recalculates each tour's start/end date from its locations. |
| `UpdateTourDays` | Recalculates each tour's total days from its locations. |

**Trigger:** `update_participant_no` (before insert on `booking`) sets `participant_no` to the number of existing bookings for that tour + 1.

---

## 8. REST API reference

Base URL: `http://localhost:3000`. All request/response bodies are JSON.

### Auth – `/api/auth`

| Method | Path | Body | Notes |
|---|---|---|---|
| POST | `/api/auth/register` | `{ email, password, role, additionalData? }` | `role`: `administrator`, `client`, `tour_operator`, `tour_guide`. `additionalData`: `{ level, clientType, toType, tgDescription }` depending on role. 409 if the e-mail exists. |
| POST | `/api/auth/login` | `{ email, password }` | Returns `{ message, userId, username, role }`. No token/session is issued. |

### Tours – `/api/tours`

| Method | Path | Description |
|---|---|---|
| GET | `/api/tours` | All tours (with average rating) |
| GET | `/api/tours/search?name=&minPrice=&maxPrice=&sortBy=` | `sortBy`: `name_asc`, `name_desc`, `price_asc`, `price_desc`, `rating_asc`, `rating_desc` |
| POST | `/api/tours` | Create tour. Required: `name, price, capacity, days, date_begin, date_end`. Optional: `description, tour_guide_id, tour_operator_id, activities, images, locations, accommodations, vehicles` |
| PUT | `/api/tours/:tour_id` | Update tour (`{ tourData, activities, images, locations, accommodations, vehicles }`) |
| DELETE | `/api/tours/:tour_id` | Delete tour |
| GET / POST / DELETE | `/api/tours/:tour_id/activities` | List / add (`{ activity }`) / remove (`{ activity }`) |
| GET / POST / DELETE | `/api/tours/:tour_id/images` | List / add (`{ image }`) / remove (`{ image }`) |
| GET | `/api/tours/:tour_id/locations` | Locations of a tour |
| PUT | `/api/tours/:tour_id/locations/:location_id` | Link / update a location |
| DELETE | `/api/tours/:tour_id/locations` | Unlink locations (`{ location_id: [..] }`) |
| GET | `/api/tours/:tour_id/accommodations` | Accommodations of a tour |
| PUT | `/api/tours/:tour_id/accommodations/:accommodation_id` | Link / update an accommodation |
| DELETE | `/api/tours/:tour_id/accommodations` | Unlink accommodations |
| GET | `/api/tours/:tour_id/vehicles` | Vehicles of a tour |
| PUT | `/api/tours/:tour_id/vehicles/:vehicle_id` | Link / update a vehicle |
| DELETE | `/api/tours/:tour_id/vehicles` | Unlink vehicles |

### Reviews – `/api/reviews`

| Method | Path | Body / query |
|---|---|---|
| GET | `/api/reviews/:tour_id?order=ASC\|DESC` | Reviews of a tour |
| POST | `/api/reviews` | `{ tour_id, client_id, rating, comment }` |
| DELETE | `/api/reviews/:review_id` | Delete a review |

### Users – `/api/users`

| Method | Path | Description |
|---|---|---|
| GET / PUT | `/api/users/:user_id/contact` | Contact details |
| GET / PUT | `/api/users/:user_id/tour-guide` | Tour guide description |
| GET / PUT | `/api/users/:user_id/client` | Client type |
| GET / PUT | `/api/users/:user_id/tour-operator` | Operator type |
| GET / PUT | `/api/users/:user_id/admin` | Admin level |
| POST | `/api/users/:client_id/bookings` | Create booking, body `{ tour_id }` (status starts as `Pending`) |
| GET | `/api/users/:client_id/bookings` | List a client's bookings |
| PUT | `/api/users/:booking_id/bookings` | Update booking status, body `{ status }` (`Confirmed`, `Cancelled`, `Pending`) |
| POST | `/api/users/:client_id/cancellation` | Create cancellation, body `{ booking_id, refund_amount, cancel_status }` |
| GET | `/api/users/:client_id/cancellation` | List a client's cancellations |
| POST | `/api/users/:client_id/payments` | Create payment, body `{ payment_date, payment_method, client_id, booking_id }` (amount is taken from the tour price; status starts as `Pending`) |
| GET | `/api/users/:client_id/payments` | List a client's payments |
| PUT | `/api/users/:payment_id/payments` | Update payment status, body `{ status: "Completed" \| "Cancelled" }` |

Example (PowerShell/CMD users: replace `\` line breaks and quotes as needed, or use Postman):

```bash
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"me@example.com","password":"12345678","role":"client","additionalData":{"clientType":"individual"}}'
```

---

## 9. Known issues / limitations

Documented honestly so contributors know where help is needed.

**Setup-related**

1. `express` and `cors` are used by `index.js` but missing from `package.json` dependencies (install them manually, see Step 2).
2. No npm script starts the backend; use `node index.js`.
3. Sample users in `booking_tour.sql` store plain-text passwords, so they fail login (see Step 5).
4. The backend URL `http://localhost:3000/` is hard-coded in `Login.jsx` and `Register.jsx`.

**Frontend still tied to a different project (a course-selling site)**

5. Home "popular tours", tour search, tour detail, cart, forget-password and create-tour pages call `https://curcus-3-0.onrender.com` (see `src/constants/baseAPI.js`, `src/configs/ApiConfig.js`, `src/apis/*`). That is a third-party demo server and not this project's backend, so these pages may show no data or fail. They need to be rewritten to call `/api/tours`, `/api/users/...` etc.
6. Many component/file names still use "course", "lesson", "student", "instructor" wording.
7. `src/apis/Cart.js` `Login`/`Register` helpers point to `https://api.example.com` (placeholders, not used by the real login page).
8. `ImageUpload.jsx` calls `data.append("upload_preset")` without a value; a Cloudinary upload preset must be added for uploads to work.

**Backend**

9. No JWT/session: login only returns the user's id and role, and the API does **not** verify who is calling. Anyone who can reach the server can read/modify data. Add authentication/authorization before any public deployment.
10. `PUT /api/users/:booking_id/bookings` fails: `bookingController.updateBookingStatus` reads `req.params.booking_id` with wrong destructuring (`const { booking_id } = req.params.booking_id`).
11. Some `user.route.js` paths overlap and the second one is never reached: `POST /:booking_id/cancellation` (shadowed by `POST /:client_id/cancellation`) and `GET /:booking_id/payments` (shadowed by `GET /:client_id/payments`).
12. `Register.jsx` collects a `name` field that the API ignores (the e-mail is used as the user name).
13. Automated tests are configured (Jest) but there are no test files yet.

---

## 10. Other npm scripts

| Command | What it does |
|---|---|
| `npm run dev` | Start the frontend dev server (hot reload) |
| `npm run build` | Create the production build in `dist/` |
| `npm run preview` | Serve the production build locally to test it |
| `npm run lint` | Run ESLint on all `.js`/`.jsx` files |
| `npm run unit_test` | Run Jest unit tests (none written yet) |
| `node index.js` | Start the backend API |

---

## 11. Troubleshooting

| Problem | Fix |
|---|---|
| `Cannot find package 'express'` (or `'cors'`) | `npm install express cors` |
| `Error connecting to MySQL: Access denied for user ...` | Wrong `MYSQL_USER` / `MYSQL_PASSWORD` in `.env` |
| `Unknown database 'booking_tour'` | Create the database first (Step 3) |
| `ECONNREFUSED 127.0.0.1:3306` | MySQL server is not running, or `MYSQL_PORT` is wrong |
| Import fails near `DELIMITER` / `CREATE PROCEDURE` | Import with the `mysql` CLI, Workbench *Data Import*, or phpMyAdmin Import (not by pasting into a single-statement box) |
| `EADDRINUSE: address already in use :::3000` | Another program uses port 3000: stop it or change `PORT` (and the URLs in `Login.jsx` / `Register.jsx`) |
| Browser console shows a CORS error | Set `CORS_ORIGIN` in `.env` to your frontend URL and restart the backend |
| Login says "Invalid credentials" for a sample user | Sample passwords are not hashed, see Step 5 |
| `bcrypt` fails to install on Windows | Use Node 18/20 LTS; if it still fails install "Desktop development with C++" via Visual Studio Build Tools, then re-run `npm install` |
| Changes to `.env` have no effect | Stop and restart `node index.js` / `npm run dev` |
| Blank page / data missing on Home or Tours | Those pages still call the external demo API, see [Known issues](#9-known-issues--limitations) |

---

## 12. Deployment

- **Frontend:** configured for [Vercel](https://vercel.com/docs/frameworks/vite) (`vite-plugin-vercel`, `vercel.json` rewrites all routes to `/` for the single-page app). Set the `VITE_*` variables in the Vercel project settings.
- **Backend / database:** host `index.js` on any Node host (Render, Railway, a VPS, ...) with a managed MySQL database, and set the same `MYSQL_*`, `PORT` and `CORS_ORIGIN` environment variables there. Update the hard-coded `http://localhost:3000/` URLs in the frontend to the deployed API URL first.
- Before going public: add authentication, restrict `CORS_ORIGIN`, and never commit `.env` files or real credentials.
