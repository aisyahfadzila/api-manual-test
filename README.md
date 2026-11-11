# Task 1 – Manual API Test (Postman)

Test cases: Google Sheet `Test_Cases_Petstore_API` (TC-01 – TC-20): https://docs.google.com/spreadsheets/d/1Spo-WQaxZ60EktuHneQNnwt9hNioZbcMHwuvKKenJns. Expected results follow the Swagger documentation.

Automated versions of these cases: `petstore-api-automation` repository (Task 2).

## Files
- `postman/petstore.postman_collection.json` – 20 requests, one per test case, each with assertions in the Tests tab.
- `postman/petstore.postman_environment.json` – `baseUrl` variable.
- `EXECUTION_REPORT.md` – fill in Actual / Status after the run.
- `BUG_REPORT.md` – one entry per failing case that is a real defect or doc mismatch.

## Run in Postman
1. Import the collection and the environment, select the **Petstore - Public** environment.
2. Run the collection (Runner). Each request creates its own pet id, so order does not matter.
3. Copy Actual + Status for each TC into the sheet and `EXECUTION_REPORT.md`. (The sheet's Summary tab updates from the Status column.)

## Run from CLI (optional)
```bash
npx newman run postman/petstore.postman_collection.json \
  -e postman/petstore.postman_environment.json \
  -r cli,htmlextra --reporter-htmlextra-export newman-report.html
```
(`npm i -g newman newman-reporter-htmlextra` first; or use plain `-r cli,json`.)

## Notes
- Assertions encode the *documented* behaviour. A failed assertion = actual result differs from the docs → candidate bug.
- TC-07 (duplicate id) is undocumented, so it only asserts "no 5xx"; record the observed status in the report.
