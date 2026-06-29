# Lab 1 — Build the Incident Coordinator in Copilot Studio

## Part 3 — Build the Agent Flow (15 min)

Everyone rejoins here. You will build an Agent Flow that fires after the agent classifies an incident — posting a Teams adaptive card, creating a Dataverse record, and routing critical incidents.

---

### Step 1 — Create a new Agent Flow

1. In the **left navigation panel** of Copilot Studio (the sidebar on the far left), click **Workflows**.
2. Click **New workflow**.
3. Name the flow. Click the default name at the top of the designer (it may show "Untitled flow" or similar). Type:
   ```
   Incident Routing and Notification
   ```
4. Click **Save**.

---

### Step 2 — Add input parameters to the trigger

1. Click on the **Start** trigger node (the first card at the top). Change the trigger type to **Connector** and select **When an agent calls the flow**.
2. The side pane appears on the right with the **Trigger inputs** section. You will add inputs one by one — these are the values the agent will pass to the flow.
3. Click **+ Add an input**. Select **Text**. A field row appears:
   - **Input name:** `Severity`
   - **Description:** `The severity level from the incident assessment: Critical, High, Medium, or Low`
4. Click **+ Add an input** again. Select **Text**.
   - **Input name:** `EscalationPath`
   - **Description:** `The team or role who should own this incident`
5. Click **+ Add an input** again. Select **Text**.
   - **Input name:** `Classification`
   - **Description:** `The incident type: Infrastructure, Application, Security, Data, User Error, or Unknown`
6. Click **+ Add an input** again. Select **Text**.
   - **Input name:** `ManagementSummary`
   - **Description:** `The 2-3 sentence plain-language management summary`
7. Click **+ Add an input** again. Select **Text**.
   - **Input name:** `IncidentText`
   - **Description:** `The original incident report submitted by the user`

---

### Step 3 — Add a condition for severity routing

1. Click the **+** (plus) button after **Start** node to add a new action.
2. In the search box that appears, select the **If/Else** control.
3. A condition card appears.
4. Iin the **Conditione** field click the lightning bolt icon (dynamic content) that appears. From the dynamic content panel, select **Severity** (the input parameter you just defined).
5. Set the **operator** dropdown to **Equals**.
6. In the **Value** field, type: `Critical`.

> This condition means: if the agent passes Severity = "Critical", take the critical escalation branch. All other severities go to the standard branch.

---

### Step 4 — Add a Teams adaptive card action (both branches)

You will add a Teams notification to both branches, with slightly different content.

#### In the "If" (Critical) branch:

1. Click the **+** button inside the **If** branch.
2. Search for **Post card in a chat or channel** (Microsoft Teams connector).
3. A connection prompt appears if you haven't connected Teams before. Click **Sign in** and authorise with your Microsoft account. Click **Allow** when prompted.
4. Configure the action:
   - **Post as:** Select **Flow bot**
   - **Post in:** Select **Channel**
   - **Team:** Select your workshop team or any team available in your tenant (e.g. *"General"* channel of any team)
   - **Channel:** Select the available channel
   - **Adaptive Card:** Click the field and paste the following JSON card template (from [adaptive-card-template.json](./assets/adaptive-card-template.json) in the workshop repo, or paste this directly):

   > **Shortcut for the workshop:** To save time, you can use the **simpler Teams action** instead of a full adaptive card. Click **+** → search for **"Post a message in a chat or channel"** → configure it with dynamic content. The adaptive card is the production pattern; the simple message works the same for lab purposes.

   **Simpler alternative message text:**
   ```
   🔴 CRITICAL INCIDENT
   Classification: [Classification dynamic content]
   Escalation: [EscalationPath dynamic content]
   Summary: [ManagementSummary dynamic content]
   ```

   To insert dynamic content: click inside the message field, then click the lightning bolt icon and select the relevant input parameter from the list.

#### In the "Else" (non-Critical) branch:

1. Click the **+** button inside the **Else** branch.
2. Add the same **Post a message in a chat or channel** action.
3. Configure with:
   - Same Team and Channel as above
   - **Message:**
   ```
   📋 Incident classified: [Classification] — Severity: [Severity]
   Assigned to: [EscalationPath]
   Summary: [ManagementSummary]
   ```
   Replace each `[...]` with the corresponding dynamic content input.

---

### Step 5 — Add a Dataverse record action

1. After the condition block (after both branches merge back), click the **+** below the condition to add an action at the bottom of the main flow path.

   > **Note:** In the flow designer, after a condition both branches eventually rejoin. Place the Dataverse action in the main flow after the condition, so it runs regardless of which branch was taken.

2. Search for **"Add a new row"** and select **Add a new row (Microsoft Dataverse)**.
3. A connection setup appears. If this is your first Dataverse connection, click **Sign in** — it connects automatically using your current account.
4. Configure:
   - **Table name:** Select **Incidents** from the dropdown.

   > **If "Incidents" table does not appear:** The table does not exist yet in your environment. Create the table and then go back to this step.

   - Map the columns:
     - **Name / Title:** Click the field, then click the lightning bolt icon → select **IncidentText** (just the first 100 characters for the title)
     - **Severity (text):** → select **Severity** dynamic content
     - **Classification (text):** → select **Classification** dynamic content
     - **Escalation Path (text):** → select **EscalationPath** dynamic content
     - **Management Summary (multiline text):** → select **ManagementSummary** dynamic content

---

### Step 6 — Add Respond to agent node

1. Add a new node - **Respond to agent**.
2. Click **Save**.

---

### Step 7 — Save and publish the workflow

1. Click **Save** in the top-right corner of the workflow designer. Wait for the save confirmation.
2. Click **Publish** to make the workflow available to the agent.

   > Workflow must be published before the agent can use them. A draft flow will not appear in the agent's tool list.

3. After publishing, click the **← Back** button (top-left) to return to your agent.

---

### Step 8 — Connect the flow as an agent tool

1. Back in the agent, click the **Tools** in the right navigation bar.
3. In the window that opens, click **Workflow**.
4. A list of available published workflows appears. You should see **"Incident Routing and Notification"** in the list.
5. Click on it to select it.
6. The workflow should be added in the Tools list.
7. Click on the workflow title in the *Tools** section to open the configuration pane. Fill in:

   **Name:**
   ```
   Route and notify incident
   ```

   **Description** (critical — the agent uses this to decide when to invoke the flow):
   ```
   Call this tool after classifying an incident report. Use it to notify the operations team via Teams, log the incident in the system, and route it to the correct escalation path. Always call this tool after producing an incident assessment.
   ```

   > **Why the description matters so much:** Generative Orchestration reads this description to decide when to invoke the flow. If it's vague ("does stuff with incidents"), the agent may not invoke it reliably. The instruction "Always call this tool after producing an incident assessment" makes the trigger explicit.

8. Under **Inputs**, for each input parameter the workflow expects, set the **How is this filled?** as `AI`.

9. Click **Save**.





