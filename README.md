# HustleHub+ Secure Freelance Marketplace (MERN)

HustleHub+ is a full-stack MERN marketplace where Freelancers list services (gigs) and Clients browse and book them through a simulated payment. Every booking creates a transaction record, and each freelancer's income is tracked from those transactions.

## Features

* Register and log in as a Client or Freelancer (JWT authentication)
* Browse and search gigs by keyword and category
* **Freelancers:** create, update and delete their own gigs; view bookings received and total income
* **Clients:** book gigs with a simulated payment; view their bookings and transactions
* Each booking and its transaction are saved together in a single MongoDB transaction

## Security Measures

| Control | Implementation |
|---------|----------------|
| Authentication | JWT (HS256, 1 h expiry) required on all protected routes; bcrypt password hashing (12 rounds); strong password policy |
| Authorisation (RBAC) | Role middleware: only Freelancers manage gigs and view income, only Clients book. Ownership checks stop users modifying or viewing others' data (403). Registering as `Admin` is blocked |
| Validation & sanitisation | `express-validator` on all inputs; XSS tags stripped with `xss`; NoSQL operators (`$`) removed and Mongoose `sanitizeFilter` enabled; 10 KB body limit |
| Rate limiting | Login/register: 20 failed attempts / 15 min; bookings: 10 / 15 min per user; all API routes: 300 / 15 min |
| Security headers | Helmet with a strict Content Security Policy, HSTS, `nosniff`, frame protection; CSP on the React app; CORS restricted to the frontend |
| Safe errors | Generic error messages with no stack traces; login does not reveal whether an email exists; booking price taken from the server |
| Transport & secrets | HTTPS (TLS); secrets kept in `.env` (git-ignored), with `.env.example` provided |

## Running the Application

**Prerequisites:** Node.js 20+, a MongoDB Atlas database, OpenSSL.

**Backend**
```bash
cd api
npm install
mkdir certs && cd certs
openssl req -nodes -new -x509 -keyout server.key -out server.cert -days 365 -subj "/CN=localhost"
cd ..
cp .env.example .env   # add your MONGO_URI and JWT_SECRET
npm run dev            # https://localhost:5001
```

**Frontend** (second terminal)
```bash
cd client
npm install
npm run dev            # http://localhost:5173
```

## Testing

**Backend: Postman + Newman.** The collection is at `api/tests/postman/HustleHub.postman_collection.json`. It covers authentication, gigs, bookings, transactions, RBAC (401/403), XSS and NoSQL injection, security headers and rate limiting. Run it with the API running:
```bash
cd api
npm run test:api        # main suite + HTML report in api/tests/reports/
npm run test:ratelimit  # shows 429 responses (run last, it blocks logins for 15 min)
```

**Frontend: Vitest + React Testing Library.** Tests component rendering and user interaction (login, registration, browsing gigs, booking, protected routes).
```bash
cd client
npm test
```

## Screenshots

**Postman**

| Collection | Collection run results |
|---|---|
| ![Postman collection](./evidence/evidence_part2/postman-collection.png) | ![Postman run results](./evidence/evidence_part2/postman-run-results.png) |

**Newman**

| Test run (CLI) | HTML report |
|---|---|
| ![Newman results](./evidence/evidence_part2/newman-results.png) | ![Newman report](./evidence/evidence_part2/newman-report.png) |
| **Rate limiting (429)** | **Rate limiting HTML report** |
| ![Rate limit](./evidence/evidence_part2/newman-ratelimit.png) | ![Rate limit report](./evidence/evidence_part2/newman-ratelimit-report.png) |

**Frontend tests (Vitest)**

![Vitest](./evidence/evidence_part2/vitest-results.png)

**Application**

| Register | Login | Browse gigs |
|---|---|---|
| ![Register](./evidence/evidence_part2/register.png) | ![Login](./evidence/evidence_part2/login.png) | ![Browse gigs](./evidence/evidence_part2/browse-gigs.png) |

| Create gig | Booking (simulated payment) | Booking confirmed |
|---|---|---|
| ![Create gig](./evidence/evidence_part2/create-gig.png) | ![Booking](./evidence/evidence_part2/purchse-booking.png) | ![Booking confirmed](./evidence/evidence_part2/booking-confirmed.png) |

| Client bookings | Freelancer dashboard (income) | Access denied (RBAC) |
|---|---|---|
| ![Client bookings](./evidence/evidence_part2/client-bookings.png) | ![Dashboard](./evidence/evidence_part2/freelancer-dashboard.png) | ![Access denied](./evidence/evidence_part2/access-denied.png) |

## Demonstration Video

[Watch the demonstration video](https://youtu.be/3Hc0sNXJRpc)
