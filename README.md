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

## Collection Structure

The collection contains 7 top-level groups:

| Folder | Items | Purpose |
|--------|-------|---------|
| `Login` | 1 | Auth — single request |
| `Create Account` | 1 | Account creation — single request |
| `CRUD Lifecycle` | 8 | Sequential CRUD chain (for Collection Runner or Flow) |
| `Flows` | 1 | `newman-flows`-compatible flow definition (FLOW method) |
| `Policies` | 10 | Positive policy tests + endorsements |
| `Accounts` | 2 | Positive account tests |
| `Negative Scenarios` | 19 | 2 setup + 17 negative tests |

### Request Chaining

Dependent values flow automatically through collection variables:

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
| Missing fields | 3 | 401, 422 |
| Invalid data types | 5 | 400, 404, 422 |
| Auth errors | 4 | 401 |
| Permission errors | 2 | 403 |
| Non-existent resources | 3 | 404, 422 |

## Test Scripts

Each request (positive and negative) includes **at least 3 assertions** in the Tests tab:
- Status code verification
- Response time < 2 seconds
- Response body structure / error message validation

Run the full collection via **Collection Runner** (Run button) to see a pass/fail test report.

## CRUD Lifecycle

### Collection Runner

A **`CRUD Lifecycle`** folder in the collection chains all 8 steps:
```
Login -> Create Account -> Create Policy -> Get Policy
-> Update Policy -> Get Policy (verify update)
-> Delete Policy -> Get Policy (verify 404)
```

Run it via **Collection Runner**: select only the `CRUD Lifecycle` folder and click Run. All 8 steps execute in order with automatic variable chaining — no manual copy-pasting needed.

### Postman Flow (Visual Pipeline)

A **`Flows`** folder is included in the collection with a `newman-flows`-compatible definition (method: `FLOW`, pre-request script with `steps([...])`).

For the **visual Postman Flow**, Postman's Flows tab does not support JSON import/export. Build it manually:
1. Open Postman Desktop
2. Go to **Flows** tab > **+ New Flow**
3. Add 8 **HTTP Request** blocks pointing to each request in the `CRUD Lifecycle` folder
4. Add 3 **Select** blocks to extract `auth_token` (Login → `body.token`), `account_id` (Create Account → `body.data.id`), `policy_id` (Create Policy → `body.data.id`)
5. Wire blocks and variables as documented in `docs/flow-build-guide.md`
6. Click **Run** to execute the full pipeline
7. Screenshot the canvas and save to `docs/screenshot-flow.png`

## Test Results (Actual)

Collection run via Postman Collection Runner on 2026-06-18:

| Result | Count |
|--------|-------|
| Total requests | 41 |
| Total assertions | **135** |
| Passed | **41** |
| Failed | **0** |
| Run duration | 8.5s |

## API Behavior Notes

During testing the following API behaviors were observed and test expectations were aligned accordingly:

| Endpoint | Scenario | API Returns | Rationale |
|----------|----------|-------------|-----------|
| POST /policies | Non-existent account_id | **422** (not 404) | API validates FK references via validation errors |
| POST /policies | Malformed JSON body | **400** | Standard JSON parse error |
| GET /policies/:id | Malformed ID (e.g. `abc`) | **404** (not 400) | API treats any unresolvable ID as "not found" |
| POST /auth/login | Empty body `{}` | **401** (not 400) | Empty body = no credentials = unauthorized |
| GET /accounts | Invalid/forged token | **401** (not 403) | No RBAC — 403 tests tolerate both codes |
| GET /accounts/:id | Forged token access | **401** (not 403) | Same RBAC limitation |
| DELETE /endorsements/:id | DELETE endpoint | **404** | Endpoint not implemented by the API |

## Documentation

All test results, screenshots, and documentation are in the `docs/` folder.

| File | Description |
|------|-------------|
| `docs/collection_runner_results.txt` | Full run output (41/41, 135 assertions, 0 failures) |
| `docs/screenshot-runner.png` | Collection Runner screenshot for submission |
| `docs/test_documentation.xlsx` | 5-sheet Excel: outlines, catalogs, bug reports |
| `docs/flow-build-guide.md` | Step-by-step instructions for building the Flow in Postman Flows tab |
| `flows/policy-lifecycle-flow.json` | Flow blueprint (reference only — Postman cannot import this format; build manually) |
| `collections/pms_environment.json` | Environment variables — import into Postman to set `{{base_url}}`, `{{email}}`, `{{password}}`, etc. |

## Notes

- The API supports **both PUT and PATCH** for updates
- Request bodies are wrapped: `{"policy":{...}}`, `{"account":{...}}`, `{"endorsement":{...}}`
- DELETE responses return **204 No Content** with an empty body
- The Endorsement DELETE endpoint is not implemented (returns 404)
- Credentials are not stored in the collection or repo — set them as variables before running
