# PMS API - Postman Collection & Documentation

Postman collection and Excel documentation for CRUD operations on the Policy Management System API.

## Credentials

Set `email` and `password` as Postman collection variables or environment variables before running.  
*A test account with rotated credentials is recommended — login details are not stored in the repo.*

## Base URL

`https://pms.srv1505121.hstgr.cloud/api/v1`

## Variables

| Variable                  | Description                          |
| ------------------------- | ------------------------------------ |
| `{{base_url}}`            | API base URL (set automatically)     |
| `{{auth_token}}`          | JWT token (set by Login request)     |
| `{{created_account_id}}`  | Account ID (set by Create Account)   |
| `{{created_policy_id}}`   | Policy ID (set by Create Policy)     |
| `{{created_endorsement_id}}` | Endorsement ID (set by Create Endorsement) |
| `{{neg_account_id}}`      | Temp account ID for negative tests (set by setup request) |
| `{{neg_policy_id}}`       | Temp policy ID for negative tests (set by setup request) |
| `{{non_existent_id}}`     | Fake UUID for 404 tests (`00000000-0000-0000-0000-000000000000`) |
| `{{forged_token}}`        | Fake JWT for permission/403 tests (set automatically) |
| `{{wrong_email}}`         | Test fixture email for wrong-credentials test (set automatically) |
| `{{wrong_password}}`      | Test fixture password for wrong-credentials test (set automatically) |

## API Endpoints

### Auth
| Method | Endpoint | Description |
| ------ | -------- | ----------- |
| POST   | `/auth/login` | Authenticate and receive JWT token |

### Account
| Method | Endpoint | Description |
| ------ | -------- | ----------- |
| POST   | `/accounts` | Create a new account |
| GET    | `/accounts` | List all accounts |
| DELETE | `/accounts/:id` | Delete an account |

### Policy (Primary Entity)
| Method | Endpoint | Description |
| ------ | -------- | ----------- |
| POST   | `/policies` | Create a policy |
| GET    | `/policies` | List all policies |
| GET    | `/policies/:id` | Get policy by ID |
| PUT    | `/policies/:id` | Full replacement (task requirement — API supports it) |
| PATCH  | `/policies/:id` | Update policy (full or partial) |
| DELETE | `/policies/:id` | Delete a policy |

### Endorsement (Secondary Entity)
| Method | Endpoint | Description |
| ------ | -------- | ----------- |
| POST   | `/policies/:policy_id/endorsements` | Create an endorsement |
| GET    | `/policies/:policy_id/endorsements` | List endorsements for a policy |
| DELETE | *(not supported — API returns 404)* | Delete endorsement |

## Request Chaining

The collection is ordered so that dependent values flow automatically:

```
Login → Create Account → Create Policy → Get All Policies
→ Get Policy By ID → Update Policy (Full) → Update Policy (Partial)
→ Update Policy (PUT) → Create Endorsement → Get All Endorsements
→ Delete Endorsement → Delete Policy → Get All Accounts → Delete Account
```

## Negative Scenarios (Week 5)

The `Negative Scenarios` folder contains **2 setup requests** (create fresh account + policy for test isolation) followed by **17 negative test requests** covering:

| Category | Count | Status Codes Tested |
|----------|-------|-------------------|
| Missing fields | 2 | 422 |
| Invalid data types | 4 | 400, 422 |
| Auth errors | 5 | 401 |
| Permission errors | 2 | 403 |
| Non-existent resources | 4 | 404, 422 |

## Test Scripts

Each request (positive and negative) includes **at least 3 assertions** in the Tests tab:
- Status code verification
- Response time < 2 seconds
- Response body structure / error message validation

Run the full collection via **Collection Runner** (Run button) to see a pass/fail test report.

## Postman Flow

A one-click **"Policy CRUD Lifecycle"** flow chains all 8 steps:
```
Login -> Create Account -> Create Policy -> Get Policy -> Update Policy
-> Get Policy (verify) -> Delete Policy -> Get Policy (verify 404)
```

Build instructions: `collections/policy_crud_flow.md`

## Test Results (Actual)

Collection run via Postman Collection Runner on 2026-06-17:

| Result | Count |
|--------|-------|
| Total requests | 33 |
| Total assertions | **107** |
| Passed | **33** |
| Failed | **0** |
| Run duration | 6.1s |

## API Behavior Notes

During testing the following API behaviors were observed and test expectations were aligned accordingly:

| Endpoint | Scenario | API Returns | Rationale |
|----------|----------|-------------|-----------|
| POST /policies | Non-existent account_id | **422** (not 404) | API validates FK references via validation errors |
| POST /policies | Malformed JSON body | **400** | Standard JSON parse error |
| POST /policies | Non-existent account_id | **422** (not 404) | API validates FK references via validation errors |
| GET /policies/:id | Malformed ID (e.g. `abc`) | **404** (not 400) | API treats any unresolvable ID as "not found" |
| POST /auth/login | Empty body `{}` | **401** (not 400) | Empty body = no credentials = unauthorized |
| GET /accounts | Invalid/forged token | **401** (not 403) | No RBAC — 403 tests tolerate both codes |
| GET /accounts/:id | Forged token access | **401** (not 403) | Same RBAC limitation |
| DELETE /endorsements/:id | DELETE endpoint | **404** | Endpoint not implemented by the API |

## Documentation

Full collection runner results are saved at `docs/collection_runner_results.txt` (33/33 passing, 107 assertions, 0 failures).

`docs/test_documentation.xlsx` contains five sheets:
1. **Test Outlines** — 12 positive test cases with expected results
2. **Request Catalog** — 14 positive requests with cURL commands
3. **Negative Outlines** — 17 negative test cases covering 5 categories
4. **Negative Request Catalog** — 17 negative requests with cURL commands, actual results, and bug comments
5. **Bugs** — 3 filed bugs (BUG-001 through BUG-003) for API discrepancies (RBAC missing, empty error body)

## Notes

- The API supports **both PUT and PATCH** for updates
- Request bodies are wrapped: `{"policy":{...}}`, `{"account":{...}}`, `{"endorsement":{...}}`
- DELETE responses return **204 No Content** with an empty body
- The Endorsement DELETE endpoint is not implemented (returns 404)
- Credentials are not stored in the collection or repo — set them as variables before running
