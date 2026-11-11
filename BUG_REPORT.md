# Bug Report – Petstore API

Environment for all bugs: https://petstore.swagger.io/v2, Postman Runner, 2026-11-11.
Expected results are taken from the Swagger documentation. Screenshots are in `evidence/`.

| ID | Endpoint | Title | Test case | Severity |
|---|---|---|---|---|
| BUG-001 | POST /pet | Empty body accepted with 200 | TC-03 | Medium |
| BUG-002 | POST /pet | Non-numeric `id` returns 500 | TC-04 | High |
| BUG-003 | POST /pet | `status` outside enum accepted with 200 | TC-05 | Medium |
| BUG-004 | GET /pet/findByStatus | Invalid `status` returns 200 instead of 400 | TC-12 | Medium |
| BUG-005 | GET /pet/findByStatus | Missing `status` returns 200 instead of 400 | TC-13 | Low |
| BUG-006 | DELETE /pet/{petId} | Non-numeric id returns 404 and leaks a Java exception | TC-18 | Medium |

---

## BUG-001 – Empty body accepted with 200
| Field | Value |
|---|---|
| Test case | TC-03 |
| Endpoint | `POST /pet` |
| Severity / Priority | Medium / Medium |

**Steps to reproduce**
1. Send `POST https://petstore.swagger.io/v2/pet` with `Content-Type: application/json`
2. Body: `{}`

**Expected:** `405 Invalid input`

**Actual:** `200 OK`, a pet is created without the required `name`:
```json
{ "id": 9223372036854561661, "photoUrls": [], "tags": [] }
```

**Evidence:** `evidence/BUG-001-TC-03.png`

**Impact:** The API stores records that violate its own schema (`name` and `photoUrls` are required).

---

## BUG-002 – Non-numeric `id` returns 500
| Field | Value |
|---|---|
| Test case | TC-04 |
| Endpoint | `POST /pet` |
| Severity / Priority | High / Medium |

**Steps to reproduce**
1. Send `POST https://petstore.swagger.io/v2/pet` with `Content-Type: application/json`
2. Body: `{ "id": "abc", "name": "qa-bad-id", "photoUrls": [] }`

**Expected:** `405 Invalid input`

**Actual:** `500 Internal Server Error`
```json
{ "code": 500, "type": "unknown", "message": "something bad happened" }
```

**Evidence:** `evidence/BUG-002-TC-04.png`

**Impact:** Invalid client input causes an unhandled server error instead of a validation error, and the message gives the client nothing to act on.

---

## BUG-003 – `status` outside the enum is accepted
| Field | Value |
|---|---|
| Test case | TC-05 |
| Endpoint | `POST /pet` |
| Severity / Priority | Medium / Medium |

**Steps to reproduce**
1. Send `POST https://petstore.swagger.io/v2/pet` with `Content-Type: application/json`
2. Body: `{ "name": "qa-bad-status", "photoUrls": [], "status": "xyz" }`

**Expected:** `405 Invalid input` (allowed values: `available`, `pending`, `sold`)

**Actual:** `200 OK`, the pet is stored with the invalid status:
```json
{ "id": 9223372036854561662, "name": "qa-bad-status", "photoUrls": [], "tags": [], "status": "xyz" }
```

**Evidence:** `evidence/BUG-003-TC-05.png`

**Impact:** Pets with an undefined status are stored, and the status enum in the documentation is not enforced. See also BUG-004.

---

## BUG-004 – Invalid `status` returns 200 instead of 400
| Field | Value |
|---|---|
| Test case | TC-12 |
| Endpoint | `GET /pet/findByStatus` |
| Severity / Priority | Medium / Medium |

**Steps to reproduce**
1. Send `GET https://petstore.swagger.io/v2/pet/findByStatus?status=xyz`

**Expected:** `400 Invalid status value`

**Actual:** `200 OK`. The response contains the pet created in BUG-003, which means the endpoint filters by any free-text value instead of validating it:
```json
[ { "id": 9223372036854561662, "name": "qa-bad-status", "photoUrls": [], "tags": [], "status": "xyz" } ]
```

**Evidence:** `evidence/BUG-004-TC-12.png`

**Impact:** Clients cannot tell a typo in the status from a valid query. Combined with BUG-003, invalid data is both stored and retrievable.

---

## BUG-005 – Missing `status` returns 200 instead of 400
| Field | Value |
|---|---|
| Test case | TC-13 |
| Endpoint | `GET /pet/findByStatus` |
| Severity / Priority | Low / Low |

**Steps to reproduce**
1. Send `GET https://petstore.swagger.io/v2/pet/findByStatus` with no query parameter

**Expected:** `400` (the `status` parameter is required)

**Actual:** `200 OK` with an empty array:
```json
[]
```

**Evidence:** `evidence/BUG-005-TC-13.png`

**Impact:** The documented required parameter is not enforced, and an empty list looks like "no pets found".

---

## BUG-006 – Non-numeric id returns 404 and leaks a Java exception
| Field | Value |
|---|---|
| Test case | TC-18 |
| Endpoint | `DELETE /pet/{petId}` |
| Severity / Priority | Medium / Low |

**Steps to reproduce**
1. Send `DELETE https://petstore.swagger.io/v2/pet/abc`

**Expected:** `400 Invalid ID supplied`

**Actual:** `404 Not Found`
```json
{ "code": 404, "type": "unknown", "message": "java.lang.NumberFormatException: For input string: \"abc\"" }
```

**Evidence:** `evidence/BUG-006-TC-18.png`

**Impact:** A malformed id is reported as "not found" instead of a client error. The raw Java exception message is exposed, which reveals internal implementation details.

---

## Test run overview
`evidence/postman-runner-summary.png`: 28 assertions, 22 passed, 6 failed.
