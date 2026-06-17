# Policy CRUD Lifecycle - Postman Flow

## How to Build This Flow in Postman

1. Open Postman, go to **Flows** tab (left sidebar or File > New Flow)
2. Click **"Create a new Flow"** and name it `Policy CRUD Lifecycle`
3. Drag the following **Send Request** blocks onto the canvas:

### Step 1: Login
- Block type: **Send Request**
- Select: `Login` from the collection
- Output: `response.body.token` -> save to collection variable `auth_token`
- Connect: Start -> this block

### Step 2: Create Account
- Block type: **Send Request**
- Select: `Create Account`
- Output: `response.body.data.id` -> save to collection variable `created_account_id`
- Connect: Login -> this block

### Step 3: Create Policy
- Block type: **Send Request**
- Select: `Create Policy`
- Output: `response.body.data.id` -> save to collection variable `created_policy_id`
- Connect: Create Account -> this block

### Step 4: Get Policy by ID
- Block type: **Send Request**
- Select: `Get Policy by ID`
- Verify: response status is 200
- Connect: Create Policy -> this block

### Step 5: Update Policy (full)
- Block type: **Send Request**
- Select: `Update Policy (full)`
- Verify: insurance_type changed to professional_liability
- Connect: Get Policy -> this block

### Step 6: Get Policy (verify update)
- Block type: **Send Request**
- Select: `Get Policy by ID`
- Verify: response body shows the updated fields
- Connect: Update Policy -> this block

### Step 7: Delete Policy
- Block type: **Send Request**
- Select: `Delete Policy`
- Verify: status is 204
- Connect: Get Policy (verify) -> this block

### Step 8: Get Policy (verify deletion)
- Block type: **Send Request**
- Select: `Get Policy by ID`
- Verify: status is **404** (confirms deletion)
- Connect: Delete Policy -> this block

### Final Step: Add Start Trigger
- Drag a **Start** block and connect it to Step 1

## Connecting Blocks

- Hover over the output dot of each block and drag to the input dot of the next block
- For variable passthrough: use the **Set Variable** block or configure the Send Request block's "Save Response" settings

## Running

Click **Run** (play button) at the top of the flow. All 8 steps execute in order automatically.

No manual copy-pasting needed - collection variables handle all data flow.
