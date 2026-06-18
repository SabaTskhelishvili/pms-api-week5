# Manual Postman Flow Build Guide

**Why manual?** Postman Flows does not support JSON import/export (confirmed by Postman staff as recently as Feb 2026 — feature "coming in a couple of weeks"). The file `flows/policy-lifecycle-flow.json` is a reference blueprint only.

Build the Flow manually in the Postman Flows tab following these steps. Screenshot the final canvas for submission.

---

## Step 1: Open the Flows Tab

1. Open Postman Desktop app
2. Click the **Flows** tab on the left sidebar
3. Click **+ New Flow** to create a new blank flow
4. Name it: `Policy CRUD Lifecycle Flow`

---

## Step 2: Add a Start Block

1. From the block palette on the left, drag a **Start** block onto the canvas

---

## Step 3: Add the 8 Request Blocks

For each of these, drag an **HTTP Request** block from the palette:

| # | Block Name | Request |
|---|-----------|---------|
| 1 | Login | Select from your collection: `CRUD Lifecycle > Login` |
| 2 | Create Account | `CRUD Lifecycle > Create Account` |
| 3 | Create Policy | `CRUD Lifecycle > Create Policy` |
| 4 | Get Policy by ID | `CRUD Lifecycle > Get Policy by ID` |
| 5 | Update Policy (full) | `CRUD Lifecycle > Update Policy (full)` |
| 6 | Get Policy by ID (verify update) | `CRUD Lifecycle > Get Policy by ID (verify update)` |
| 7 | Delete Policy | `CRUD Lifecycle > Delete Policy` |
| 8 | Get Policy by ID (verify deletion) | `CRUD Lifecycle > Get Policy by ID (verify deletion)` |

How to select a request from your collection:
- After dragging in an HTTP Request block, click on it
- In the right panel, click the dropdown under **Select Request**
- Navigate to the collection > `CRUD Lifecycle` folder > pick the request
- The URL, method, headers, and body auto-populate from the collection

---

## Step 4: Add 3 Select Blocks (Variable Extraction)

Drag **Select** blocks from the palette to extract dynamic IDs:

### Select 1: Extract auth_token (from Login)
- Connect the Select block's input to Login's success output
- Set expression: `body.token`
- Rename the block: `Extract auth_token`

### Select 2: Extract account_id (from Create Account)
- Connect input from Create Account's success output
- Set expression: `body.data.id`
- Rename: `Extract account_id`

### Select 3: Extract policy_id (from Create Policy)
- Connect input from Create Policy's success output
- Set expression: `body.data.id`
- Rename: `Extract policy_id`

---

## Step 5: Wire the Blocks

Create connections by clicking-and-dragging from output ports (right side) to input ports (left side):

### Trigger chain (arrows going into top ports):

```
Start  -->  Login  -->  Extract auth_token (data input)
                  -->  Create Account (trigger input)
                             -->  Extract account_id (data input)
                             -->  Create Policy (trigger input)
                                        -->  Extract policy_id (data input)
                                        -->  Get Policy by ID (trigger input)
                                                   -->  Update Policy (full) (trigger input)
                                                              -->  Get Policy by ID (verify update) (trigger input)
                                                                         -->  Delete Policy (trigger input)
                                                                                    -->  Get Policy by ID (verify deletion) (trigger input)
```

### Variable wiring (variables ports on the side):

| Source | Target Variable Port |
|--------|---------------------|
| Extract auth_token output | Create Account `auth_token` |
| Extract auth_token output | Create Policy `auth_token` |
| Extract auth_token output | Get Policy by ID `auth_token` |
| Extract auth_token output | Update Policy (full) `auth_token` |
| Extract auth_token output | Get Policy by ID (verify update) `auth_token` |
| Extract auth_token output | Delete Policy `auth_token` |
| Extract auth_token output | Get Policy by ID (verify deletion) `auth_token` |
| Extract account_id output | Create Policy `created_account_id` |
| Extract policy_id output | Get Policy by ID `created_policy_id` |
| Extract policy_id output | Update Policy (full) `created_policy_id` |
| Extract policy_id output | Get Policy by ID (verify update) `created_policy_id` |
| Extract policy_id output | Delete Policy `created_policy_id` |
| Extract policy_id output | Get Policy by ID (verify deletion) `created_policy_id` |

---

## Step 6: Run the Flow

1. Click the **Run** button (play icon) in the top bar
2. Watch the blocks turn green as each executes
3. If any block turns red, click it to see the error details

**Expected result:** All 8 request blocks execute in sequence, with variables chained automatically. No manual copy-pasting needed.

---

## Step 7: Screenshot

1. Make sure all blocks and wires are visible (zoom out with Ctrl+scroll if needed)
2. Take a screenshot showing the entire canvas
3. Save as `docs/screenshot-flow.png` alongside the runner screenshot

---

## Reference: Blueprint File

The file `flows/policy-lifecycle-flow.json` contains the complete block/connection layout in Postman's internal format (schema: `https://schema.getpostman.com/json/flow/v1/flow.json`). This is for reference only — it cannot be imported by Postman, but it documents every block, position, port, and connection in case you need to rebuild it.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Variable port not appearing on block | Click the block, check that the variable name matches exactly (e.g. `auth_token`, `created_account_id`, `created_policy_id`) |
| "Select Request" shows nothing | Make sure the collection is saved and synced to your workspace |
| Block turns red on run | Click the block to see the error — most common: missing variable value, or token expired |
| Wrong variable value | Check the Select block expression path — use Postman's response viewer to find the correct path |
