# Test Execution Report – Petstore API

- **Tester:** Aisyah Fadzila Hani
- **Base URL:** https://petstore.swagger.io/v2
- **Tool:** Postman Runner (collection `postman/petstore.postman_collection.json`, environment "Petstore - Public")
- **Run date:** 2026-11-11, 21:18 – 1 iteration, 9s 857ms, avg response time 361 ms
- **Assertions:** 28 total, 22 passed, 6 failed, 0 errors

Expected results follow the Swagger documentation. Bug IDs refer to `BUG_REPORT.md`; response screenshots are in `evidence/`.

## POST /pet
| ID | Scenario | Expected | Actual | Status | Bug |
|---|---|---|---|---|---|
| TC-01 | All valid fields | 200, response matches request | 200, response matches request | Pass | |
| TC-02 | Required fields only | 200 | 200 | Pass | |
| TC-03 | Empty body | 405 | 200, pet created without `name` (`{"id":9223372036854561661,"photoUrls":[],"tags":[]}`) | **Fail** | BUG-001 |
| TC-04 | Wrong type for id | 405 | 500, `"something bad happened"` | **Fail** | BUG-002 |
| TC-05 | Status outside enum | 405 | 200, pet stored with `"status": "xyz"` | **Fail** | BUG-003 |
| TC-06 | XML content type | 200 | 200 | Pass | |
| TC-07 | Duplicate id | Behavior noted (no 5xx) | Request handled without a server error (status below 500) | Pass | |

## GET /pet/findByStatus
| ID | Scenario | Expected | Actual | Status | Bug |
|---|---|---|---|---|---|
| TC-08 | available | 200, all available | 200, all available | Pass | |
| TC-09 | pending | 200, all pending | 200, all pending | Pass | |
| TC-10 | sold | 200, all sold | 200, all sold | Pass | |
| TC-11 | available,sold | 200, only those | 200, only available or sold | Pass | |
| TC-12 | Invalid status (`xyz`) | 400 | 200, array containing the pet with status `xyz` | **Fail** | BUG-004 |
| TC-13 | Missing parameter | 400 | 200, `[]` | **Fail** | BUG-005 |
| TC-14 | New pet is listed | id present | id present | Pass | |

## DELETE /pet/{petId}
| ID | Scenario | Expected | Actual | Status | Bug |
|---|---|---|---|---|---|
| TC-15 | Delete existing | 200, message = petId | 200, message = petId | Pass | |
| TC-16 | Delete twice | 404 | 404 | Pass | |
| TC-17 | Non-existent | 404 | 404 | Pass | |
| TC-18 | Non-numeric id (`abc`) | 400 | 404, `java.lang.NumberFormatException: For input string: "abc"` | **Fail** | BUG-006 |
| TC-19 | Gone after delete | 404 Pet not found | 404 Pet not found | Pass | |
| TC-20 | With api_key header | 200 | 200 | Pass | |

## Summary
| Endpoint | Total | Pass | Fail | Blocked | Pass rate |
|---|---|---|---|---|---|
| POST /pet | 7 | 4 | 3 | 0 | 57% |
| GET /pet/findByStatus | 7 | 5 | 2 | 0 | 71% |
| DELETE /pet/{petId} | 6 | 5 | 1 | 0 | 83% |
| **Total** | 20 | 14 | 6 | 0 | **70%** |

## Conclusion
The happy paths of all three endpoints work (create, filter by each status, delete, and the delete-then-get flow). Every failure is in negative or validation handling: the API does not follow its documented error codes. It accepts invalid input (empty body, status outside the enum), returns 500 for a type error where the docs say 405, returns 200 instead of 400 for an invalid or missing `status`, and returns 404 instead of 400 for a non-numeric id. Impact: clients cannot rely on the documented status codes for error handling, and bad data can be stored.
