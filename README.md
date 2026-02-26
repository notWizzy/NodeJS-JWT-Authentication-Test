# NodeJS JWT Authentication Test

A minimal demo app showing **JWT authentication** in Node.js using **Express**.

- **Backend:** Express API that issues JWTs on login and protects routes with `express-jwt`.
- **Frontend:** A single `index.html` page that logs in, stores the token in `localStorage`, and calls protected endpoints.

## Project structure

```
.
├── server.js        # Express server + JWT auth middleware
├── index.html       # Simple UI (Axios) to login + call protected routes
├── package.json
└── package-lock.json
```

## Prerequisites

- Node.js + npm

## Run locally

```bash
npm install
node server.js
```

Then open:

- http://localhost:3000

The server listens on port **3000** and serves `index.html` from `/`. fileciteturn1file0L18-L19 fileciteturn1file0L88-L90

## Demo users

Users are hard-coded in `server.js`: fileciteturn1file0L25-L36

- `fabio` / `123`
- `nolasco` / `456`

> Note: due to the current login loop implementation, only the first matching check reliably works (see **Known issues**). fileciteturn1file0L41-L61

## How authentication works

### 1) Login → receive a JWT

`POST /api/login` expects JSON:

```json
{ "username": "fabio", "password": "123" }
```

If credentials match, the server signs a JWT with payload `{ id, username }` and an expiry of **3 minutes**. fileciteturn1file0L38-L47

Example:

```bash
curl -s -X POST http://localhost:3000/api/login   -H "Content-Type: application/json"   -d '{"username":"fabio","password":"123"}'
```

Response:

```json
{
  "success": true,
  "err": null,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### 2) Call protected routes with `Authorization: Bearer <token>`

The app uses `express-jwt` middleware configured with **HS256** and the same secret used to sign tokens. fileciteturn1file0L19-L23

Protected endpoints:

- `GET /api/dashboard` fileciteturn1file0L64-L70
- `GET /api/prices` fileciteturn1file0L72-L78
- `GET /api/settings` fileciteturn1file0L80-L86

Example:

```bash
TOKEN="<paste token here>"
curl -s http://localhost:3000/api/dashboard   -H "Authorization: Bearer $TOKEN"
```

If the token is missing/invalid/expired, the server returns **401 Unauthorized**. fileciteturn1file0L92-L99

## Frontend behavior (`index.html`)

- On login, the browser saves the token to `localStorage` under the key `jwt`, then loads the dashboard. fileciteturn1file1L38-L50
- Protected API calls include the JWT via the `Authorization: Bearer ...` header. fileciteturn1file1L53-L59
- The page decodes the JWT payload and checks for expiry periodically, redirecting back to `/` when it detects expiration. fileciteturn1file2L60-L89

## Known issues / gotchas

These are present in the current code as uploaded:

1. **Login loop returns 401 too early**  
   The `else` clause inside the `for` loop sends a 401 response on the first non-match, which prevents checking the rest of the users list. fileciteturn1file0L41-L61

2. **CORS header typo**  
   `Access-Control-Allow-Headers` includes `Aythorization` (typo). If you call this API cross-origin, you'll want `Authorization` instead. fileciteturn1file0L12-L15

3. **Token expires in 3 minutes, but UI checks every 4 minutes**  
   Token expiry is set to `3m`. fileciteturn1file0L43-L47  
   The UI checks for expiry every `240000` ms (4 minutes). fileciteturn1file2L81-L90

## Next ideas (optional)

- Move users to a database and hash passwords (`bcrypt`)
- Store the JWT secret in environment variables (`process.env.JWT_SECRET`)
- Add refresh tokens + logout
- Add tests (Jest / Supertest)

---

### Quick API reference

| Method | Path | Auth | Description |
|---|---|---:|---|
| POST | `/api/login` | No | Returns a JWT if credentials match |
| GET | `/api/dashboard` | Yes | Example protected resource |
| GET | `/api/prices` | Yes | Example protected resource |
| GET | `/api/settings` | Yes | Example protected resource |
| GET | `/` | No | Serves `index.html` |
