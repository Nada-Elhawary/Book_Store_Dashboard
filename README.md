# NexusBooks — Book Rental Platform

A full-stack book rental web application built with Next.js 16 and Express 5. Users can browse a catalog, rent and return books, and track their rental history. Admins manage the catalog and monitor all platform activity through a dedicated dashboard.

---

## Overview

NexusBooks is a MERN-stack application with a clear frontend/backend separation. The backend exposes a REST API (deployed as a Vercel serverless function) and the frontend is a Next.js App Router application. Authentication is handled with JWT tokens stored in `localStorage`, and role-based access distinguishes regular users from admins.

The project was built as a portfolio piece demonstrating full-stack development across data modeling, REST API design, authentication/authorization, and a component-based React UI.

---

## Features

### Authentication & Authorization
- User registration and login with hashed passwords (bcryptjs)
- JWT-based authentication with a 30-day token lifetime
- Token attached via `Authorization: Bearer` header on every protected request
- Two roles: `user` (default) and `admin`
- Protected routes enforced on both the API (middleware) and the frontend (client-side auth guard)

### Public Catalog
- Browse all available books without an account
- Client-side search/filter by title or author
- Availability indicator per book (copies remaining)

### User Functionality
- Rent an available book (decrements copy count atomically)
- View personal rental history with status (rented / returned)
- Return a book (increments copy count, timestamps the return)
- Duplicate rental prevention — cannot rent the same book twice while it is active

### Admin Functionality
- Full CRUD for the book catalog (add, edit, delete)
- View all platform orders with user and book details
- Dashboard overview: total users, total books, active rentals, lifetime orders

### Dashboard
- Role-aware dashboard: separate views rendered for admin and regular user
- Recent activity feed for both roles
- Stat cards with live data fetched from the API

### UI / UX
- Responsive layout (mobile sidebar drawer, desktop persistent sidebar)
- Light/dark theme toggle, persisted in `localStorage`
- Animated landing page (Framer Motion)
- Toast notifications for all user actions (Sonner)

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Frontend framework** | Next.js 16 (App Router) |
| **Frontend language** | TypeScript |
| **UI library** | shadcn/ui (base-nova style, @base-ui/react primitives) |
| **Styling** | Tailwind CSS v4 (CSS-only config, no tailwind.config.ts) |
| **Animations** | Framer Motion |
| **State management** | Zustand |
| **Forms** | React Hook Form + Zod |
| **HTTP client** | Axios (single instance with auth interceptor) |
| **Backend framework** | Express 5 |
| **Backend language** | JavaScript (Node.js) |
| **Database** | MongoDB via Mongoose |
| **Authentication** | JSON Web Tokens (jsonwebtoken) + bcryptjs |
| **Security middleware** | Helmet, CORS, express-rate-limit |
| **Logging** | Morgan |
| **Deployment target** | Vercel (frontend + backend as serverless function) |

---

## Project Structure

```
bookstore-api/
├── backend/
│   ├── config/
│   │   └── db.js               # Mongoose connection with serverless-safe caching
│   ├── controllers/
│   │   ├── bookController.js   # CRUD operations for books
│   │   ├── orderController.js  # Rent, return, list orders
│   │   └── userController.js   # Register, login, getMe, admin helpers
│   ├── middleware/
│   │   ├── authMiddleware.js   # JWT verification + requireAdmin guard
│   │   └── errorMiddleware.js  # Global Express error handler
│   ├── models/
│   │   ├── Book.js             # title, author, image, availableCopies
│   │   ├── Order.js            # user ref, book ref, status, timestamps
│   │   └── User.js             # name, email, hashed password, role
│   ├── routes/
│   │   ├── bookRoutes.js
│   │   ├── orderRoutes.js
│   │   └── userRoutes.js
│   ├── app.js                  # Express app setup, middleware, routes
│   ├── seed.js                 # Script to populate the database with sample books
│   └── server.js               # Entry point; Vercel handler or local Express server
│
└── frontend/
    └── src/
        ├── app/
        │   ├── (auth)/
        │   │   ├── layout.tsx          # Split-panel auth layout
        │   │   ├── login/page.tsx
        │   │   └── register/page.tsx
        │   ├── books/page.tsx          # Public catalog with search
        │   ├── dashboard/
        │   │   ├── admin/
        │   │   │   ├── books/page.tsx  # Admin: manage catalog
        │   │   │   └── orders/page.tsx # Admin: all rentals
        │   │   ├── rentals/page.tsx    # User: rental history + return
        │   │   ├── layout.tsx          # Auth guard + sidebar shell
        │   │   └── page.tsx            # Role-aware dashboard overview
        │   ├── globals.css             # Tailwind v4 theme tokens (color palette, radius)
        │   └── layout.tsx              # Root layout: fonts, ThemeProvider, Toaster
        ├── components/
        │   ├── layout/
        │   │   ├── PublicHeader.tsx    # Fixed nav for public pages
        │   │   ├── PublicFooter.tsx
        │   │   └── Sidebar.tsx         # Dashboard sidebar (desktop + mobile drawer)
        │   ├── theme-provider.tsx      # Custom React context theme provider
        │   ├── theme-toggle.tsx
        │   └── ui/                     # shadcn/ui components (button, card, dialog, table, …)
        ├── lib/
        │   ├── axios.ts                # Axios instance with auth interceptor
        │   └── utils.ts                # cn() helper
        └── store/
            └── authStore.ts            # Zustand auth store (user, token, login, logout)
```

---

## API Reference

Base URL: `http://localhost:5000` (local) or your deployed Vercel URL.

All protected routes require the header:
```
Authorization: Bearer <token>
```

### Users

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/api/users/register` | — | Register a new user |
| POST | `/api/users/login` | — | Authenticate and receive a token |
| GET | `/api/users/me` | User | Get the current authenticated user |
| GET | `/api/users` | Admin | List all registered users |

### Books

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/api/books` | — | Get all books |
| GET | `/api/books/:id` | — | Get a single book |
| POST | `/api/books` | Admin | Add a new book |
| PATCH | `/api/books/:id` | Admin | Update a book |
| DELETE | `/api/books/:id` | Admin | Delete a book |

### Orders

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/api/orders/rent/:bookId` | User | Rent a book |
| PUT | `/api/orders/return/:orderId` | User | Return a rented book |
| GET | `/api/orders/my-orders` | User | Get the current user's orders |
| GET | `/api/orders` | Admin | Get all orders across all users |

---

## Data Models

### User
```
name           String   required
email          String   required, unique
password       String   required, hashed with bcryptjs (salt rounds: 10)
role           String   enum: "user" | "admin", default: "user"
```

### Book
```
title             String   required
author            String   required
image             String   URL, optional (defaults to a placeholder)
availableCopies   Number   default: 1
```

### Order
```
user         ObjectId   ref: User, required
book         ObjectId   ref: Book, required
status       String     enum: "rented" | "returned", default: "rented"
rentedAt     Date       default: Date.now
returnedAt   Date       set on return
```

---

## Getting Started

### Prerequisites

- Node.js ≥ 18
- npm ≥ 9
- A MongoDB instance (local or [MongoDB Atlas](https://www.mongodb.com/atlas))

### Installation

**1. Clone the repository**
```bash
git clone <repository-url>
cd bookstore-api
```

**2. Install backend dependencies**
```bash
cd backend
npm install
```

**3. Install frontend dependencies**
```bash
cd ../frontend
npm install
```

### Environment Variables

**Backend** — create `backend/.env`:
```env
PORT=5000
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret_key
FRONTEND_URL=http://localhost:3000
```

**Frontend** — create `frontend/.env`:
```env
NEXT_PUBLIC_API_URL=http://localhost:5000
```

### Running Locally

**Start the backend** (from `backend/`):
```bash
npm run dev
```
The API will be available at `http://localhost:5000`.

**Start the frontend** (from `frontend/`):
```bash
npm run dev
```
The app will be available at `http://localhost:3000`.

### Seeding the Database

To populate the database with a set of sample books:
```bash
cd backend
node seed.js
```
This clears the existing `books` collection and inserts 15 sample titles.

> **Note:** There is no seed script for admin users. To create an admin, register a normal account and then manually update the `role` field to `"admin"` in your MongoDB database.

---

## Deployment

The backend is structured for deployment as a **Vercel serverless function**. `server.js` exports an Express handler that Vercel invokes per request. The MongoDB connection is cached on the `global` object to avoid creating a new connection on every cold start.

The frontend is a standard Next.js application and deploys to Vercel with no additional configuration.

Both services need their respective environment variables set in the Vercel project settings.

---

## Known Limitations

- **No email verification.** Accounts are activated immediately on registration.
- **No password reset.** There is no forgot-password flow.
- **No pagination.** All books and orders are returned in a single response.
- **No due-date enforcement.** The `Order` model stores `returnedAt` but the API does not calculate or enforce due dates. The admin orders page references a `dueDate` field that is not present in the current schema.
- **Client-side auth guard only.** Dashboard route protection is done in a `useEffect` on the client; there is no server-side middleware.
- **Admin accounts created manually.** There is no UI or API endpoint to promote a user to admin.
