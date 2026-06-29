# Lab 1 — Build the Incident Coordinator in Copilot Studio

## Part 1 — Create the agent and configure knowledge (25 min)

### Step 1 — Open Copilot Studio and select your environment

1. Open your browser (Chrome or Edge).
2. Navigate to **https://copilotstudio.preview.microsoft.com/**
3. Sign in with the Microsoft account you set up during prerequisites.
4. In the **top-right corner**, look at the environment selector. It shows the name of your current Power Platform environment (e.g. *"Alex's environment"*).
5. If it does not show your developer or trial environment, click the environment name → a dropdown appears → select your environment from the list.

> **Why this matters:** Agents are stored inside environments. If you build in the wrong environment, you cannot find your agent later. Use your developer environment throughout all labs.

---

### Step 2 — Create a new agent

1. In the **left navigation panel**, click **Agents**.
2. Hit **Create Agent** to start crafting your first Copilot Studio agent. Make sure you are in the **New experience**.

3. The **New agent** form opens. Fill in the fields:

   **Name:** `Incident Coordinator`

   **Instructions:**  
   Open [system-prompt-lab1.txt](./assets/system-prompt-lab1.txt) from the workshop repo and copy the full contents. Paste it into the Instructions field.

4. Keep **Model** `GPT-5.3 Chat` or select another.

5. Click **Save** in the top-right corner.

---

### Step 3 — Add the incident playbook as a knowledge source

1. In the **right pane** of the agent, click plus icone next to the **Knowledge**.
2. The Knowledge page opens.
3. Click **Browse** or drag and drop [incident-playbook.pdf](./assets/incident-playbook.pdf) from your laptop into the upload area.
4. The file appears in the upload list.
5. Click **Add to agent** to confirm adding this knowledge source.

> **Note:** Indexing the PDF takes 1–3 minutes in the background. You will see the status as *"Processing"* in the Knowledge tab.

6. Once the file processed, click **+ Add knowledge** again and explore the other source types available. Do NOT add anything — just observe what's available. Click **Cancel** to close without adding.

---

### Step 4 — First test in the Test pane

1. In the **top area** of the screen, locate the **Preview** button. Click it to open the Test window.
2. A chat interface appears on the screen.
3. In the message input at the bottom of the test panel, type or paste **Incident #1** from the workshop dataset:

   ```
   Our payment service is throwing 503 errors intermittently since about 14:30. Started after the deployment. Not sure if it's the new build or the infra. Clients are complaining. Logs attached but I haven't had time to look at them.
   ```

4. Press **Enter** or click the send button (paper plane icon).
5. Wait for the agent response. It should appear within 3-5 seconds.

**What to check in the response:**
- Does it follow the structured format defined in the instructions?
- Does it include all 9 fields (Classification, Severity, Urgency, Business impact, Risk score, Escalation path, Missing information, Recommended action, Management summary)?
- Is the classification reasonable for a payment service 503 error?

> **Expected:** Classification = Application or Infrastructure. Severity = High or Critical. Risk score = 7–9.

6. If the format is missing fields or looks like free text, look at the **Knowledge** tab — the PDF may still be processing. Wait 60 seconds and try again.

7. Now test **Incident #4** — the deceptively calm one:

   ```
   Hi, I noticed the nightly backup job didn't complete last night. Probably fine but thought I should mention it. The job runs at 2am. Let me know if you need anything.
   ```

**What to check:**
- Does the agent catch that a missed backup is not "probably fine"?
- Does it assign appropriate severity despite the casual tone of the reporter?
- Does Missing information note that we need to know how many backup cycles have been missed?

> **This is a deliberate test.** The incident is written to sound minor but is potentially critical (data loss risk). Observe whether the agent reasons correctly or is misled by the reporter's casual tone. Note your observation on the Assessment Card.

