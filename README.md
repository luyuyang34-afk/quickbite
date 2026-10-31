# QuickBite — Frontend

React + TypeScript single-page app for **QuickBite**, a DoorDash-style food ordering website.

The backend is a Spring Boot REST API in a separate repo: [OnlineOrder](https://github.com/luyuyang34-afk/OnlineOrder). In production, this app is built and served by the backend from the same origin.

## Features

- Browse restaurants. Search, filter by category, and sort.
- Save favorite restaurants (stored in `localStorage`) and view them on `/saved`.
- View a restaurant's menu and add dishes to the cart.
- Sign up, log in, and log out with a server-side session cookie.
- Cart, checkout, and order confirmation pages. These are protected routes: logged-out users are sent to `/login?redirect=...` and come back after logging in.
- Loading skeletons, error states, toast notifications, and a 404 page.

## Tech Stack

| Area | Technology |
|---|---|
| UI | React 19, TypeScript |
| Build | Vite 8 |
| Routing | React Router 7, with lazy-loaded pages |
| Server state | TanStack Query 5: caching, refetching, invalidation after mutations |
| Styling | Tailwind CSS 4, Radix UI primitives with shadcn-style components, lucide icons |
| Feedback | sonner (toasts) |
| Lint | oxlint |

## Project Structure

```
src/
├── main.tsx              # App entry: QueryClient, Router, SessionProvider
├── App.tsx               # Route table
├── pages/
│   ├── discover.tsx      # Home: restaurant list, search, saved restaurants
│   ├── restaurant.tsx    # Restaurant menu, add to cart
│   ├── auth.tsx          # Login and signup
│   └── cart.tsx          # Cart, checkout, order confirmation
├── components/           # Layout, ProtectedRoute, restaurant card, ui/ (button, dialog, input...)
└── lib/
    ├── api.ts            # fetch wrapper for every backend call
    ├── queries.ts        # TanStack Query hooks (useRestaurants, useCart, useAddToCart...)
    ├── session.tsx       # Login state shared across the app
    ├── favorites.ts      # Saved restaurants in localStorage
    └── types.ts          # Shared TypeScript types
```

## How It Talks to the Backend

- Every request goes through `lib/api.ts`. It uses `fetch` with `credentials: "include"`, so the browser sends the `JSESSIONID` cookie. Each request has a 15-second timeout. HTTP errors (401, 403, 429...) become user-friendly messages.
- Login is sent as form-urlencoded data, which is what Spring Security form login expects. Other request bodies are JSON with snake_case keys, such as `{"menu_id": 1}`.
- To check whether the user is logged in, the app calls `GET /cart`. A `200` means logged in; a `401` means guest.
- After adding to the cart or checking out, the cart query is invalidated, so the UI refetches the latest cart.

| User action | Backend endpoint |
|---|---|
| Open home page | `GET /restaurants/menu` |
| Open a restaurant | `GET /restaurant/{id}/menu` |
| Sign up / log in / log out | `POST /signup`, `POST /login`, `POST /logout` |
| View cart | `GET /cart` |
| Add a dish | `POST /cart` |
| Checkout | `POST /cart/checkout` |

## Run Locally

Prerequisites: Node.js 20.19+, plus the [backend](https://github.com/luyuyang34-afk/OnlineOrder) running on `http://localhost:8080`.

```bash
npm install
npm run dev   # http://localhost:5173
```

In development, the app calls `/api/...`. The Vite dev server proxies those requests to `http://localhost:8080` and strips the `/api` prefix. Set `API_PROXY_TARGET` to proxy somewhere else.

| Script | Description |
|---|---|
| `npm run dev` | Start the dev server with hot reload |
| `npm run build` | Type-check and build to `dist/` |
| `npm run preview` | Preview the production build |
| `npm run lint` | Run oxlint |

## Build and Deploy

```bash
npm run build
```

`.env.production` sets `VITE_API_BASE_URL=/`, so the built app calls the API on the same origin.

To deploy:

1. Copy the contents of `dist/` into the backend's `src/main/resources/public/`.
2. Rebuild and deploy the backend image (see the [OnlineOrder README](https://github.com/luyuyang34-afk/OnlineOrder#deploy-to-aws)).

One container then serves both the frontend and the API, so no CORS setup is needed.
