# Internship-Applicant-Management-API

A REST API for managing internship applicants built with **Node.js + Express** and **sql.js** (pure JavaScript SQLite). No external database installation required — works out of the box.

---

## Tech Stack

| Layer      | Technology                        |
|------------|-----------------------------------|
| Runtime    | Node.js 18+                       |
| Framework  | Express 5                         |
| Database   | sql.js (SQLite — pure JavaScript) |
| Testing    | Jest + Supertest                  |

---

## Project Structure

```
internship-api-v2/
├── server.js                    # Entry point
├── src/
│   ├── app.js                   # Express app & middleware
│   ├── db/
│   │   └── database.js          # sql.js database wrapper
│   ├── middleware/
│   │   └── validation.js        # Input validation
│   └── routes/
│       └── candidates.js        # All 5 API endpoints
├── tests/
│   ├── testHelper.js            # In-memory DB for test isolation
│   └── candidates.test.js       # 23 automated tests
├── data/
│   └── internship.sqlite        # Auto-created on first run
└── package.json
```

---

## Getting Started

### Prerequisites
- Node.js **18 or higher**
- npm

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/YOUR_USERNAME/internship-api.git
cd internship-api

# 2. Install dependencies
npm install

# 3. Start the server
npm start
```

Server runs at: **http://localhost:3000**

### Run Tests

```bash
npm test
```

---

## API Endpoints

### Base URL
```
http://localhost:3000
```

### Response Format
```json
{ "success": true,  "data": { } }
{ "success": false, "errors": [ ] }
```

---

### Root
**GET** `/`

Returns a welcome message and all available endpoints.

```json
{
  "success": true,
  "message": "Welcome to Internship Applicant Management API",
  "version": "1.0.0",
  "endpoints": {
    "health":           "GET    /health",
    "createCandidate":  "POST   /candidates",
    "getAllCandidates":  "GET    /candidates",
    "getCandidateById": "GET    /candidates/:id",
    "updateStatus":     "PUT    /candidates/:id/status",
    "deleteCandidate":  "DELETE /candidates/:id"
  }
}
```

---

### 1. Create Candidate
**POST** `/candidates`

**Request Body:**
```json
{
  "fullName": "Ali Hassan",
  "email": "ali@gmail.com",
  "university": "FAST NUCES",
  "cgpa": 3.5,
  "status": "Applied"
}
```

> `status` is optional and defaults to `Applied`

**Response `201`:**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "fullName": "Ali Hassan",
    "email": "ali@gmail.com",
    "university": "FAST NUCES",
    "cgpa": 3.5,
    "status": "Applied",
    "createdAt": "2026-06-18T14:00:00.000Z",
    "updatedAt": "2026-06-18T14:00:00.000Z"
  }
}
```

| Code | Reason |
|------|--------|
| `201` | Candidate created successfully |
| `400` | Validation failed |
| `409` | Email already exists |

---

### 2. Get All Candidates
**GET** `/candidates`

**Optional Query Parameters:**

| Param        | Description             | Example               |
|--------------|-------------------------|-----------------------|
| `status`     | Filter by status        | `?status=Shortlisted` |
| `university` | Partial name search     | `?university=FAST`    |
| `sortBy`     | Field to sort by        | `?sortBy=cgpa`        |
| `order`      | `asc` or `desc`         | `?order=desc`         |

**Response `200`:**
```json
{
  "success": true,
  "count": 2,
  "data": [ { ... }, { ... } ]
}
```

---

### 3. Get Candidate By ID
**GET** `/candidates/:id`

**Example:** `GET /candidates/1`

**Response `200`:**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "fullName": "Ali Hassan",
    "email": "ali@gmail.com",
    "university": "FAST NUCES",
    "cgpa": 3.5,
    "status": "Applied"
  }
}
```

| Code | Reason |
|------|--------|
| `200` | Candidate found |
| `404` | Candidate not found |

---

### 4. Update Candidate Status
**PUT** `/candidates/:id/status`

**Example:** `PUT /candidates/1/status`

**Request Body:**
```json
{
  "status": "Shortlisted"
}
```

**Valid Status Values:**

| Status        | Description           |
|---------------|-----------------------|
| `Applied`     | Just applied          |
| `Shortlisted` | Selected for review   |
| `Interviewed` | Interview completed   |
| `Selected`    | Offer given           |
| `Rejected`    | Not selected          |

**Response `200`:**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "status": "Shortlisted"
  }
}
```

| Code | Reason |
|------|--------|
| `200` | Status updated |
| `400` | Invalid or missing status |
| `404` | Candidate not found |

---

### 5. Delete Candidate
**DELETE** `/candidates/:id`

**Example:** `DELETE /candidates/1`

**Response `200`:**
```json
{
  "success": true,
  "message": "Candidate deleted successfully."
}
```

| Code | Reason |
|------|--------|
| `200` | Deleted successfully |
| `404` | Candidate not found |

---

### Health Check
**GET** `/health`

```json
{
  "status": "ok",
  "timestamp": "2026-06-18T14:00:00.000Z"
}
```

---

## Candidate Schema

| Field        | Type    | Required | Description                            |
|--------------|---------|----------|----------------------------------------|
| `id`         | integer | Auto     | Auto-incremented (1, 2, 3...)          |
| `fullName`   | string  | Yes      | Non-empty string                       |
| `email`      | string  | Yes      | Valid format, unique, stored lowercase |
| `university` | string  | Yes      | Non-empty string                       |
| `cgpa`       | number  | Yes      | Between `0.0` and `4.0`               |
| `status`     | string  | No       | Defaults to `Applied`                  |
| `createdAt`  | string  | Auto     | ISO 8601 timestamp                     |
| `updatedAt`  | string  | Auto     | Updated on every change                |

---

## Validation Rules

| Field        | Rule                                          |
|--------------|-----------------------------------------------|
| `fullName`   | Required, cannot be empty                     |
| `email`      | Required, valid email format, must be unique  |
| `university` | Required, cannot be empty                     |
| `cgpa`       | Required, number between `0` and `4`          |
| `status`     | Must be one of 5 defined values               |

---

## Error Handling

| HTTP Code | Meaning                  |
|-----------|--------------------------|
| `400`     | Validation error         |
| `404`     | Record not found         |
| `409`     | Duplicate email conflict |
| `500`     | Unexpected server error  |

---

## Testing

23 automated tests covering all endpoints:

```bash
npm test
```

```
✓ POST /candidates — creates with valid data
✓ POST /candidates — returns 400 for invalid email
✓ POST /candidates — returns 409 for duplicate email
✓ GET  /candidates — returns all candidates
✓ GET  /candidates/:id — returns correct candidate
✓ GET  /candidates/:id — returns 404 for missing id
✓ PUT  /candidates/:id/status — updates status
✓ PUT  /candidates/:id/status — returns 400 for invalid status
✓ DELETE /candidates/:id — deletes candidate
✓ ... and 14 more
```

---

## Assumptions

1. **No PostgreSQL required** — uses sql.js (pure JS SQLite), works without any DB installation
2. **Numeric IDs** — auto-incremented integers instead of UUIDs for simplicity
3. **Email is unique** — one candidate per email address
4. **Email stored lowercase** — `ALI@EXAMPLE.COM` is saved as `ali@example.com`
5. **No status ordering enforced** — any valid status can be set at any time
6. **No authentication** — API is open; JWT auth would be added for production
7. **CGPA scale 0–4** — standard GPA scale
