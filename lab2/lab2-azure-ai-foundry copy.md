# Lab 2 — Build the Intelligence Layer in Microsoft Foundry

**Duration:** 75 minutes  
**Platform:** Microsoft Foundry — portal only, no code
**Azure subscription:** Required (free account with $200 credit is sufficient)  
**URL:** https://ai.azure.com

---

## What you will build

By the end of this lab you will have:

- A **Foundry project** with a GPT-4.1 deployment you control
- A **Foundry IQ knowledge base** backed by Foundry Tools (ex. Azure AI Search) — the enterprise retrieval layer that grounds your agent in the incident playbook
- A **Prompt Agent** configured with temperature 0 and enforced JSON Schema output — producing consistent, structured incident assessments every time
- A **visual Workflow** that orchestrates the agent with branching logic, all built in the portal without writing a line of code
- An **Evaluation run** against the same 10-incident dataset from Lab 1, with execution traces that show exactly where failures occur
- A guided tour of the **Foundry Control Plane** — quotas, content safety filters, RBAC, and the Agent Monitoring Dashboard

You will use the same incident scenario and the same test dataset as Lab 1.
The outputs are directly comparable. The differences you observe are the architectural lesson.

---

## Workshop resources — have these ready

| File | Used in |
|------|---------|
| `incident-playbook.pdf` | Part 2 — Foundry IQ knowledge source |
| `system-prompt-lab2.txt` | Part 3 — agent instructions |
| `json-schema-incident.json` | Part 3 — response format schema |
| `evaluation-set-10.csv` | Part 4 — same dataset as Lab 1 |

---

## Part 1 — Create your Foundry project and deploy the model (10 min)

### Step 1 — Open the Foundry portal and enable New Foundry

1. Open your browser (Chrome or Edge).
2. Navigate to **https://ai.azure.com**
3. Sign in with the Microsoft account linked to your Azure subscription.
4. You land on the Foundry home page. Look at the **top navigation bar** — 
   find the toggle labelled **"New Foundry"** (it may appear as a banner or
   a toggle near the top-right of the screen).
5. Make sure the **New Foundry toggle is ON** (blue/enabled). If it is off,
   click it to enable it.

   > **Why this matters:** The New Foundry experience is where Workflows,
   > Foundry IQ, and the Control Plane live. The classic view does not have
   > these features. Everything in this lab requires New Foundry to be on.

---

### Step 2 — Create a new Foundry project

1. On the Foundry home page, click **+ Create project** (or **Create** if
   you see a large button in the centre of the screen).
2. The project creation form opens. Fill in the fields:

   **Project name:**
   ```
   contoso-ops-hub
   ```

   **Subscription:** Select your Azure subscription from the dropdown.

   **Resource group:** Either select an existing resource group or click
   **Create new** and type:
   ```
   rg-contoso-workshop
   ```

   **Region:** Select **Sweden Central**.

   > If Sweden Central is not available in your subscription, use
   > **East US 2** as the fallback. Do NOT use Italy North or Brazil South —
   > File Search and some agent features are unavailable there.

   Leave all other fields at their defaults.

3. Click **Create**.

   > Provisioning takes 60–120 seconds. Foundry creates a Foundry resource,
   > a storage account, and sets up the RBAC role assignments automatically.
   > You do not need to touch the Azure portal for any of this.

4. When provisioning completes, the portal drops you directly into your
   new project. You see a project home page with a top menu bar showing
   **Build**, **Model catalog**, **Evaluate**, **Manage**, and
   **Control Plane**.

---

### Step 3 — Deploy GPT-4.1 from the model catalog

1. In the top menu bar of your project, click **Model catalog**.
2. The model catalog opens showing hundreds of models. In the search box
   at the top, type:
   ```
   gpt-4.1
   ```
3. The results filter. Click on the **GPT-4.1** card (published by Azure
   OpenAI).
4. On the model detail page, click the blue **Deploy** button.
5. A deployment configuration panel opens. Fill in:

   **Deployment name:**
   ```
   gpt41-incident
   ```

   **Deployment type:** Select **Global Standard**

   **Tokens per minute (TPM):** Set to **30,000** (or higher if your quota
   allows — do not exceed your available quota)

   Leave all other settings at their defaults.

6. Click **Deploy**.

   > Deployment takes 30–60 seconds. When complete, you see a success
   > notification and the model appears in your **Models + endpoints** list.

7. Click **Open in playground** on the success screen to verify the
   deployment is working. Type `Hello` in the chat — you should receive a
   response. Close the playground tab when done.

   > **First architectural contrast with Lab 1:** In Copilot Studio, you
   > could not choose your model deployment, name it, or set its token limit.
   > You just used "GPT-4.1" — an abstraction. Here, you own the deployment
   > configuration. That control is exactly what makes the next steps possible.

---

## Part 2 — Set up Foundry IQ knowledge base (15 min)

Foundry IQ is the enterprise knowledge layer for Foundry agents. It sits on
top of Azure AI Search and provides agentic retrieval — meaning it
automatically decomposes queries into subqueries, searches in parallel, and
applies semantic reranking before returning results to the agent.

This is the production-grade equivalent of what Copilot Studio does with
its knowledge sources — but with visible retrieval, citation tracking, and
reusable knowledge bases that any number of agents can share.

---

### Step 4 — Navigate to Foundry IQ

1. In the top menu bar, click **Build**.
2. In the left sidebar that appears, look for **Foundry IQ** (it may also
   appear as **Knowledge bases** depending on your portal version).
3. Click **Foundry IQ**.

   > You see the Foundry IQ overview page. It currently shows no knowledge
   > bases — you will create one now.

---

### Step 5 — Create an Azure AI Search resource (Free tier)

Foundry IQ requires an Azure AI Search resource as its underlying
retrieval engine. You will create a Free tier instance — sufficient for
the lab (1 index, 50 MB storage, no cost).

1. On the Foundry IQ page, click **+ Create knowledge base**.
2. A setup wizard opens. The first step asks you to connect to a search
   service. Click **Connect to an AI Search resource**.
3. A panel shows your existing search services. If you have none, click
   **Create new resource**. This opens the Azure portal in a new browser
   tab.
4. In the Azure portal, you are on the **Create a search service** page.
   Fill in:

   **Subscription:** Your Azure subscription  
   **Resource group:** Select `rg-contoso-workshop` (the one you created
   in Step 2)  
   **Service name:** Type a globally unique name, for example:
   ```
   search-contoso-workshop-[your initials]
   ```
   (Replace `[your initials]` with 2–3 letters to make it unique)

   **Region:** **Sweden Central** (must match your Foundry project region)

   **Pricing tier:** Click **Change Pricing Tier** → select **Free**

5. Click **Review + create**, then click **Create**.

   > Provisioning takes 1–3 minutes. Watch the deployment progress. When it
   > shows "Your deployment is complete", click **Go to resource**.

6. Switch back to the **Foundry portal browser tab**.
7. In the search service connection panel, click the **Refresh** button
   (↻) to load the newly created search service.
8. Select your `search-contoso-workshop-[initials]` service from the list.
9. For **Auth type**, select **API key**.
10. Click **Connect**.

   > The Foundry portal now has a connection to your Azure AI Search service.

---

### Step 6 — Create the incident playbook knowledge base

1. Back on the Foundry IQ page, click **+ Create knowledge base**.
2. Fill in the knowledge base details:

   **Name:**
   ```
   incident-playbook-kb
   ```

   **Description:**
   ```
   Contoso incident response playbook — classification criteria, severity 
   definitions, escalation paths, and recommended actions for all incident types.
   ```

   **Search service:** Your `search-contoso-workshop-[initials]` service
   (should already be selected from the connection you just made)

3. Click **Next**.
4. On the knowledge sources step, click **+ Add knowledge source**.
5. A panel opens showing source types. Click **Files** (or **Upload files**).
6. Click **Browse** and select `incident-playbook.pdf` from the workshop
   repo on your laptop.
7. The file appears in the upload list. Click **Upload**.
8. Under **Retrieval reasoning effort**, select **Standard**.

   > **Retrieval reasoning effort** controls how aggressively Foundry IQ
   > decomposes and iterates on queries:
   > - **Basic** — single-pass retrieval, fastest, least thorough
   > - **Standard** — recommended for most scenarios, good balance
   > - **High** — multi-pass iterative retrieval, most thorough, uses more tokens

9. Click **Create knowledge base**.

   > Foundry IQ now automatically handles: chunking the PDF → creating
   > embeddings → indexing into Azure AI Search → making it ready for
   > agentic retrieval. This takes 2–4 minutes.

10. You see a progress indicator. The knowledge base status changes from
    **Indexing** → **Ready**.

---

### Step 7 — Test the knowledge base directly

Before connecting it to the agent, test the knowledge base in isolation.
This is a Foundry IQ capability that Copilot Studio does not expose — you
can see exactly what the retrieval layer returns before the model ever sees it.

1. Click on **incident-playbook-kb** in the knowledge base list.
2. Click **Test retrieval** (or the **Test** tab inside the knowledge base).
3. A query input field appears. Type this query:

   ```
   What is the escalation path for a Critical severity infrastructure incident?
   ```

4. Click **Search** (or press Enter).
5. The results panel shows:
   - The retrieved document chunks, ranked by relevance
   - The source section of the PDF each chunk came from
   - The relevance score for each chunk

6. Try a second query:

   ```
   How should I classify an incident where production data may have been exposed?
   ```

7. Observe: Foundry IQ decomposed this into subqueries internally. The
   results show chunks from the Security section of the playbook.

> **Observation for the Assessment Card:** You can see exactly which sections
> of the playbook will be used to ground the agent's response. In Lab 1,
> you had no visibility into this. Write down whether this visibility changes
> how confident you feel about the agent's answers.

---

## Part 3 — Build the Prompt Agent (15 min)

### Step 8 — Create a new Prompt Agent

1. In the top menu bar, click **Build**.
2. In the left sidebar, click **Agents**.
3. Click **+ New agent**.
4. A panel appears asking for the agent type. Select **Prompt agent**.

   > A Prompt Agent is a single-model agent driven entirely by a system
   > prompt and tools. It is the simplest, most controllable agent type in
   > Foundry — no orchestration framework, no hidden logic.

5. Fill in the agent details:

   **Agent name:**
   ```
   Incident Intelligence Agent
   ```

   **Model:** Click the model dropdown and select **gpt41-incident**
   (the deployment you created in Step 3).

   Leave all other fields empty for now. Click **Create**.

   > The agent is created and you are taken directly to its configuration
   > page — the Agent Studio.

---

### Step 9 — Configure the system prompt

1. In the Agent Studio, you see a **System prompt** (or **Instructions**)
   text area on the left side.
2. Clear any default text in the field.
3. Open `system-prompt-lab2.txt` from the workshop repo and copy the
   full contents. Paste it into the System prompt field.

   The system prompt reads:

   ```
   You are the Contoso Incident Intelligence Agent. Your sole function is 
   to analyse incident reports submitted by Contoso operations staff and 
   produce a structured JSON assessment.

   You must respond ONLY with a valid JSON object. Do not include any 
   explanatory text, preamble, markdown formatting, or code fences. 
   Output only the raw JSON object.

   The JSON object must contain exactly these fields:

   {
     "classification": "Infrastructure | Application | Security | Data | User Error | Unknown",
     "severity": "Critical | High | Medium | Low",
     "urgency": "Immediate | Within 2 hours | Within 8 hours | Next business day",
     "business_impact": "One sentence describing who and what is affected",
     "risk_score": 1-10,
     "escalation_path": "Specific team or role name",
     "missing_information": "List of missing details, or null if complete",
     "recommended_action": "One specific, concrete immediate action",
     "management_summary": "2-3 sentences in plain non-technical language"
   }

   Base your assessment on the incident report provided and your knowledge 
   base. Apply conservative judgment — when in doubt between two severity 
   levels, choose the higher one. A missed backup is never Low severity. 
   An automated alert with no business impact context is always missing 
   information.

   Never ask clarifying questions. Never explain your reasoning. 
   Output only the JSON object.
   ```

4. The system prompt is saved automatically as you type.

---

### Step 10 — Set temperature and response format

This is where Foundry gives you control that Copilot Studio does not.

1. Look for the **Configuration** or **Model parameters** panel — it is
   usually in the right column of the Agent Studio, or accessible via a
   **Settings** button near the model selector.
2. Find the **Temperature** slider or input field. Set it to:
   ```
   0
   ```

   > Temperature 0 makes the model deterministic — given the same input and
   > the same retrieved context, it will always produce the same output. This
   > is essential for a classification task where you need consistency.

3. Find the **Response format** setting. Click it and select
   **JSON Schema** (or **Structured output**).
4. A text field appears for the schema. Click inside it and paste the
   contents of `json-schema-incident.json` from the workshop repo:

   ```json
   {
     "type": "object",
     "properties": {
       "classification": {
         "type": "string",
         "enum": ["Infrastructure", "Application", "Security", "Data", 
                  "User Error", "Unknown"]
       },
       "severity": {
         "type": "string",
         "enum": ["Critical", "High", "Medium", "Low"]
       },
       "urgency": {
         "type": "string",
         "enum": ["Immediate", "Within 2 hours", "Within 8 hours", 
                  "Next business day"]
       },
       "business_impact": { "type": "string" },
       "risk_score": { "type": "integer", "minimum": 1, "maximum": 10 },
       "escalation_path": { "type": "string" },
       "missing_information": { "type": ["string", "null"] },
       "recommended_action": { "type": "string" },
       "management_summary": { "type": "string" }
     },
     "required": ["classification", "severity", "urgency", "business_impact",
                  "risk_score", "escalation_path", "missing_information",
                  "recommended_action", "management_summary"]
   }
   ```

5. Click **Save** (or **Apply**) to confirm the schema.

   > **What JSON Schema enforcement does:** The model is now constrained at
   > the API level to only produce output that matches this schema. It cannot
   > output freeform text, cannot skip a field, cannot invent a new severity
   > level. This is not prompt engineering — it is a hard structural
   > constraint enforced by the model runtime.

---

### Step 11 — Connect the Foundry IQ knowledge base as a tool

1. In the Agent Studio, look for the **Tools** section or a **+ Add tool**
   button (it may be in a sidebar or a tab at the bottom of the screen).
2. Click **+ Add tool**.
3. In the tool picker panel, select **Azure AI Search** (or
   **Foundry IQ / Knowledge base**).
4. A list of available knowledge bases appears. Select
   **incident-playbook-kb**.
5. Click **Add**.

   > The knowledge base now appears as a tool connected to the agent.
   > At inference time, the agent will automatically call this tool when
   > it needs context from the playbook. It uses Model Context Protocol
   > (MCP) under the hood — the same protocol that Work IQ uses in Studio.

6. **Do not add any other tools.** For this lab, the agent has only one
   tool: the Foundry IQ knowledge base. No code interpreter, no Bing
   Search, no function calls.

---

### Step 12 — Test in the Agent Playground

1. Click **Test in playground** (or switch to the **Playground** tab at
   the top of the Agent Studio).
2. The playground chat interface opens. You are now talking directly to
   your configured agent.
3. Submit **Incident #1** — the same one you used in Lab 1:

   ```
   Our payment service is throwing 503 errors intermittently since about 14:30. 
   Started after the deployment. Not sure if it's the new build or the infra. 
   Clients are complaining. Logs attached but I haven't had time to look at them.
   ```

4. Press **Enter**. The agent responds.

**What to check:**
- Is the response pure JSON with no surrounding text? (It should be.)
- Does every required field appear?
- Are enum values exactly as specified (e.g. `"High"` not `"high"` or
  `"HIGH"`)?
- Compare to your Lab 1 response — same incident, different output.
  Note any differences in classification, severity, or the management
  summary quality.

5. Now click **View trace** (or the **↗ Trace** button) next to the
   response.

   > A trace panel opens showing the full execution:
   > - The system prompt sent to the model
   > - The knowledge base tool call — including the query sent to
   >   Foundry IQ and the chunks returned
   > - The model's final response
   > - Latency at each step (ms)
   > - Token counts for input and output

   This is the visibility that Lab 1 could not give you. You can see exactly
   which paragraphs of the incident playbook were retrieved and used to
   ground the response.

6. Submit **Incident #4** — the deceptively calm backup failure:

   ```
   Hi, I noticed the nightly backup job didn't complete last night. 
   Probably fine but thought I should mention it. 
   The job runs at 2am. Let me know if you need anything.
   ```

7. View the trace. Answer these questions:
   - What query did the agent send to the Foundry IQ knowledge base?
   - What chunks were returned from the playbook?
   - Did the playbook contain relevant guidance about backup failures that
     changed the severity judgment?
   - Compare to Lab 1: did Studio catch the severity correctly? Does
     Foundry handle it differently, and can you now explain why?

8. Submit **Incident #3** — the automated monitoring alert:

   ```
   ALERT: CPU utilization on prod-db-02 exceeded 95% threshold for 
   8 consecutive minutes. Region: West Europe. Service: customer-data-api.
   ```

   In the trace, observe the missing_information field — the agent should
   note that there is no business impact context. The playbook guidance
   on automated alerts should surface in the retrieved chunks.

---

## Part 4 — Build the visual Workflow (10 min)

The Foundry Workflow is a visual, no-code orchestration builder. You define
a sequence of nodes — each node is a step. The workflow can invoke agents,
apply branching logic, and expose a REST endpoint. No YAML, no code.

This is the Foundry equivalent of the Agent Flow you built in Lab 1.
Both are visual. Both allow branching. The key difference: the Foundry
Workflow is deterministic — every step, every branch, every execution is
logged and inspectable. Compare this with Copilot Studio's generative
orchestration, which decides at runtime what to do next.

---

### Step 13 — Create a blank Workflow

1. In the top menu bar, click **Build**.
2. In the left sidebar, click **Agents**.
3. At the top of the Agents page, click the **Workflows** tab.
4. Click **+ Create** → **Blank workflow**.

   > The visual workflow builder opens. You see a canvas with no nodes.
   > On the left is a node palette. In the centre is the canvas where
   > you drag and connect nodes.

5. Name the workflow. Click the default name at the top of the builder
   (it may show "Untitled workflow"). Type:
   ```
   Incident Triage Workflow
   ```

---

### Step 14 — Add the Start node

1. On the canvas, click the **+** button (or drag a node from the palette).
2. Select **Start** from the node type list.
3. A Start node appears on the canvas. Click on it to open its configuration.
4. Add an input variable. Click **+ Add input**:
   - **Name:** `IncidentText`
   - **Type:** String
   - **Description:** `The raw incident report text submitted by the user`
5. Click **Save** on the node configuration.

---

### Step 15 — Add an Invoke Agent node

1. Click the **+** button below the Start node (or click the plus icon
   on the connector line coming out of Start).
2. Select **Invoke agent** from the node type list.
3. The Invoke Agent node configuration opens. Fill in:

   **Select agent:** Click the agent picker. Select
   **Incident Intelligence Agent** (the agent you created in Part 3).

   **Input:** Map the agent's input to the workflow variable:
   - The agent expects a user message. In the input field, click the
     dynamic variable picker (the `{}` icon or the `@` symbol).
   - Select **IncidentText** from the list of available workflow variables.

   **Output:** The agent returns a JSON string. Name this output:
   ```
   AgentResponse
   ```

4. Click **Save**.

---

### Step 16 — Add a Condition node

1. Click the **+** below the Invoke Agent node.
2. Select **Condition** (or **If/else**) from the node type list.
3. The condition configuration opens. Fill in the condition expression
   using Power Fx:

   Click in the condition field and type:
   ```
   outputs.AgentResponse.severity = "Critical"
   ```

   > Power Fx is the same formula language used in Power Apps and Power
   > Automate. You do not need to learn it — this single expression is
   > all you need for the lab.

4. The condition creates two output paths:
   - **True branch** — when severity is Critical
   - **False branch** — all other severities

---

### Step 17 — Add End nodes to both branches

#### True branch (Critical):

1. Click the **+** on the **True** output of the Condition node.
2. Select **End**.
3. Configure the End node output:
   - **Variable name:** `Triage_Result`
   - **Value:** Click the variable picker and select **AgentResponse**
   - **Priority:** Add a text field set to `"CRITICAL — Immediate escalation required"`
4. Click **Save**.

#### False branch (Standard):

1. Click the **+** on the **False** output of the Condition node.
2. Select **End**.
3. Configure:
   - **Variable name:** `Triage_Result`
   - **Value:** Select **AgentResponse**
   - **Priority:** `"Standard — Route per escalation path"`
4. Click **Save**.

---

### Step 18 — Test the Workflow step by step

1. Click **Test** (or the ▶ Play button) in the top-right of the
   workflow builder.
2. A test panel opens on the right. You see an input field for
   **IncidentText**.
3. Paste **Incident #1**:

   ```
   Our payment service is throwing 503 errors intermittently since about 14:30. 
   Started after the deployment. Not sure if it's the new build or the infra. 
   Clients are complaining. Logs attached but I haven't had time to look at them.
   ```

4. Click **Run**.
5. Watch the canvas — nodes light up as they execute:
   - **Start** node → green (input received)
   - **Invoke Agent** node → spinning (calling the Incident Intelligence Agent)
   - **Condition** node → evaluates severity
   - The appropriate **End** node → lights up with the output

6. In the test panel on the right, expand each completed node to see:
   - Its input
   - Its output
   - How long it took (latency in ms)

7. Now test with **Incident #4** (the backup failure). Observe which
   branch the condition takes — Critical or standard?

> **The contrast with Lab 1:** In Studio's Agent Flow, you could see the
> flow ran and whether Teams was notified. You could NOT step through each
> node and see exactly what each step received and returned. In Foundry's
> Workflow, every node execution is visible and inspectable at test time.
> This determinism is a governance and debugging asset.

---

### Step 19 — Note the Workflow endpoint

1. After testing, click **Deploy** (or **Publish**) in the workflow builder.
2. Once deployed, click **Settings** or **Endpoints** on the workflow page.
3. You will see the **Workflow endpoint URL** and the **API key** location.

   > **Save these — you will need them in Lab 3.** The endpoint URL and
   > API key are how Copilot Studio will call this Foundry Workflow in Lab 3.

4. You can also click **Copy endpoint** to copy the URL to your clipboard.
   Open your REST client (Bruno/Postman) and do a quick manual test:

   **Method:** POST  
   **URL:** Your workflow endpoint URL  
   **Header:** `api-key: [your primary key from Project settings → Keys and endpoints]`  
   **Header:** `Content-Type: application/json`  
   **Body:**
   ```json
   {
     "IncidentText": "The nightly backup job did not complete. Probably fine."
   }
   ```

   Send the request. You should receive the structured JSON assessment
   back as the response body. If you do — the endpoint is confirmed
   working and Lab 3 is ready.

---

## Part 5 — Run Evaluation (15 min)

The Foundry evaluation wizard runs your agent against a dataset and scores
it on multiple quality dimensions. Crucially, every failing result links
directly to the execution trace — so you can see not just *that* it failed
but *why* it failed and *where* in the execution the failure occurred.

---

### Step 20 — Open the Evaluation section

1. In the top menu bar of your project, click **Evaluate**.
2. The Evaluation page opens. Click **+ New evaluation**.
3. A wizard starts. On the first screen, select:

   **What are you evaluating?** → **Agent**  
   **Select agent:** → **Incident Intelligence Agent**

4. Click **Next**.

---

### Step 21 — Upload the evaluation dataset

1. On the dataset screen, select **Upload a file**.
2. Click **Browse** and select `evaluation-set-10.csv` from the workshop repo.

   > The CSV format Foundry expects:
   > - Column: `query` — the incident text (what the user submits)
   > - Column: `ground_truth` — the expected correct assessment (used by
   >   AI-judged evaluators to compare against)
   >
   > The workshop CSV has both columns for all 10 incidents.

3. The file uploads and Foundry shows a preview of the first few rows.
   Confirm the column mappings look correct.
4. Click **Next**.

---

### Step 22 — Select evaluators

1. On the evaluators screen, you see a list of available quality metrics.
   Enable **three** evaluators by checking their boxes:

   ✅ **Groundedness** — Is the response supported by the retrieved context
   (the playbook)? Catches hallucination — when the agent makes up facts
   not in the document.

   ✅ **Relevance** — Does the response actually address what was asked?
   Catches when the agent produces a correct-looking response that misses
   the actual question.

   ✅ **Task adherence** — Did the agent follow its instructions? Checks
   whether the output format, field completeness, and behavioural rules
   from the system prompt were followed.

2. For the AI judge model (used to score responses), leave the default
   selection. Foundry will use a strong model to evaluate your agent's
   responses against the ground truth.
3. Click **Next**.

---

### Step 23 — Configure and run

1. Give the evaluation a name:
   ```
   Incident Dataset v1 — Lab 2
   ```
2. Review the summary — 10 test cases, 3 evaluators, Incident Intelligence
   Agent.
3. Click **Run evaluation**.

   > The evaluation runs in the background. With 10 test cases and 3
   > evaluators, it typically takes 3–5 minutes. A progress bar shows
   > overall completion.

   While the evaluation runs, continue to Step 24.

---

### Step 24 — Explore a trace from the Playground

While the evaluation runs, look at one trace in detail from the manual
tests you ran earlier.

1. Click **Build** in the top menu bar → **Agents** → click on
   **Incident Intelligence Agent**.
2. In the Agent Studio, click the **Traces** tab (or **Activity**).
3. You see a list of recent inference calls from the Playground tests.
4. Click on the trace for **Incident #4** (the backup failure).
5. The trace detail view opens. Explore the three sections:

   **Section A — Input**
   - The full system prompt sent to the model (every token)
   - The user message (the incident text)
   - The knowledge base tool call request (the query sent to Foundry IQ)

   **Section B — Tool execution**
   - The query Foundry IQ received: note how it may be different from
     the raw incident text — the agentic retrieval engine decomposed it
   - The document chunks returned: you can read the actual text of each
     chunk and its relevance score
   - The order in which chunks were returned

   **Section C — Output**
   - The raw JSON response from the model
   - Token count: input tokens + output tokens
   - Latency: total ms, breakdown by step

> **This is the root cause capability:** If an evaluation result fails,
> you can link directly from the failing score to this trace and see
> whether the failure was:
> - The retrieval step (wrong chunks returned — Foundry IQ issue)
> - The model step (right chunks returned but model ignored them — prompt issue)
> - The format step (model returned correct content but wrong structure — schema issue)
>
> In Lab 1, none of this was visible. You knew the agent failed. You did
> not know why.

---

### Step 25 — Review evaluation results

1. Return to the **Evaluate** section in the top menu bar.
2. Your evaluation run should now be complete (if not, wait until it is).
3. Click on **Incident Dataset v1 — Lab 2** to open the results.
4. The results table shows:
   - Each row = one test case (one incident report)
   - Each column = one evaluator score (Groundedness / Relevance / Task adherence)
   - Each cell shows a score (typically 1–5) and a Pass/Fail indicator
5. Look at the **summary panel** on the right — it shows aggregate scores
   across all 10 test cases for each evaluator.

6. Find a **failing row** (a test case with a low score on any evaluator).
   Click on it.
7. In the detail panel that opens:
   - You see the full agent response for this test case
   - You see the evaluator's reasoning: exactly why it gave a low score
   - Click **View trace** (if available) — this links you to the trace
     for this exact inference call

8. Answer the question on your Assessment Card:
   - Was the failure a retrieval problem (wrong chunks), a model problem
     (ignored good chunks), or a format problem (wrong schema)?
   - Could you tell which one without the trace?

---

### Step 26 — Compare evaluation results with Lab 1

Look at your Lab 1 Assessment Card. Now fill in the comparison:

| Dimension | Lab 1 (Studio) | Lab 2 (Foundry) |
|-----------|----------------|-----------------|
| Output consistency | Variable — no temperature control | Consistent — temperature 0 + JSON Schema |
| Evaluation: can I see pass/fail? | Yes | Yes |
| Evaluation: can I see WHY it failed? | Limited | Yes — full trace |
| Can I see which doc chunk was used? | No | Yes — in every trace |
| Can I fix a specific failure? | Re-prompt and hope | Identify exact step, fix precisely |

---

## Part 6 — Foundry Control Plane (10 min, trainer-led)

The Control Plane is where Foundry moves from a per-project developer tool
to an enterprise governance platform. This section is a guided walkthrough
— the trainer leads, you follow along on your own screen.

---

### Step 27 — Open the Control Plane

1. In the top menu bar of the Foundry portal, click **Control Plane**.

   > If you do not see Control Plane in the top menu, look for it in the
   > left sidebar or under **Manage**. It may also be accessible from
   > the Foundry home page (before entering a specific project).

2. The Control Plane opens in a new view, showing cross-project governance
   rather than a single-project view.

---

### Step 28 — Quotas

1. In the Control Plane left sidebar, click **Quotas**.
2. You see a table of all model deployments across your account, with
   their allocated TPM (Tokens Per Minute) and usage.
3. Find your `gpt41-incident` deployment. It shows:
   - Region: Sweden Central
   - Allocated: 30,000 TPM
   - Current usage: the tokens consumed so far in the lab

> **The governance point:** An architect or AI Operations team sets quotas
> centrally here. Individual project teams cannot exceed what they're
> allocated. This is how you prevent a runaway agent from consuming
> thousands of dollars of tokens. In Copilot Studio, the equivalent
> is Copilot Credits — but the granularity and control model is different.

4. Click **Request quota increase** to see the form (do not submit — just
   explore). Observe the fields: region, model, requested TPM, business
   justification. This is the same form you used during prerequisites.

---

### Step 29 — Agent Monitoring Dashboard

1. In the Control Plane left sidebar, click **Monitoring** (or
   **Agent monitoring**).
2. The Agent Monitoring Dashboard opens. It shows:

   - **Quality metric trends** — Groundedness, Relevance, Task adherence
     scores over time (populated by the evaluation runs you just completed)
   - **Latency distribution** — how long agent calls are taking (p50, p95,
     p99 latency in ms)
   - **Token usage** — input and output tokens per day, trended over time
   - **Safety threshold status** — whether any content safety filters
     have triggered

3. Your dashboard may not show much data yet — that is expected after a
   single lab session. The trainer will show a pre-populated dashboard
   screenshot to illustrate what production monitoring looks like.

> **The key insight:** The evaluation results you generated in Step 25 flow
> directly into this dashboard. As you run more evaluations over time, this
> dashboard shows whether agent quality is improving or degrading after each
> change — instruction update, knowledge base update, model version change.

---

### Step 30 — Content safety filters

1. In the Control Plane, navigate to **Content safety** (or go back to
   your project → **Models + endpoints** → click on **gpt41-incident**).
2. You see the content filtering configuration for this model deployment.
3. The filter categories are:
   - **Hate** — hate speech and discrimination
   - **Violence** — violent content
   - **Sexual** — sexually explicit content
   - **Self-harm** — content related to self-harm
4. Each category has a **Severity threshold** slider: Low / Medium / High.
   Content at or above the threshold is blocked.
5. There is also a **Groundedness filter** setting — specific to RAG
   scenarios. When enabled, it detects when the model's response is not
   grounded in the retrieved context (i.e., hallucination detection at
   the API level).

> **The governance contrast with Lab 1:** In Copilot Studio, content safety
> is a tenant-level setting managed by Microsoft. You cannot configure
> thresholds per agent or per deployment. In Foundry, each model deployment
> has its own filter configuration. An incident triage agent with a playbook
> about security incidents may need different thresholds than a customer-
> facing agent. Here, you control that.

6. **Do not change any settings** — just observe the current configuration.

---

### Step 31 — RBAC role assignments

1. In your Foundry project, click on the project name in the top-left
   to go to the project settings (or navigate via **Manage → Access**).
2. Click **Access control (IAM)** or **Role assignments**.
3. You see the current role assignments for this project:
   - Your account should show as **Foundry Owner** (because you created
     the project)
   - The search service managed identity may show as **Search Index Data
     Contributor** (set up by Foundry IQ automatically)

4. Click **+ Add role assignment** to see the form (do not save — just
   explore):
   - **Role:** Foundry User (can invoke agents, cannot create or modify)
   - **Assign access to:** User, group, or service principal
   - **Members:** Would be your colleague's Entra ID account

> **The governance point:** A developer can be a Foundry User — they can
> call the agent endpoint but cannot change the system prompt or deploy
> a new model. A team lead is a Foundry Contributor — they can edit agents
> but not manage access. The Foundry Owner manages the project. This is
> Azure RBAC applied to AI — the same model your organisation already uses
> for every other Azure resource.

5. Click **Cancel** without saving.

---

### Step 32 — Knowledge base access control (Foundry IQ governance)

1. Navigate to **Build → Foundry IQ**.
2. Click on **incident-playbook-kb**.
3. Click the **Settings** or **Access** tab inside the knowledge base.
4. Observe: the knowledge base has its own access control, separate from
   the agent that uses it.

> **The enterprise pattern:** In production, a Data Team manages the knowledge
> base — they decide what documents go in, control who can modify it, and
> manage the Azure AI Search resource underneath it. An AI Team manages the
> agent — they write the system prompt and configure the tools. These are
> two different ownership domains at the same RBAC boundary. Both teams
> can work independently without stepping on each other.

> **The contrast with Lab 1:** In Copilot Studio, the knowledge source is
> part of the agent — the same person who builds the agent also controls
> the knowledge. There is no separate ownership boundary.

---

## Part 7 — Complete Assessment Card 2 (5 min)

Fill in every row before the Architecture Battle.

| Dimension | Your score (1–5) | Your note |
|-----------|-----------------|-----------|
| **Speed to working solution** | | Compared to Lab 1 — faster or slower, and why? |
| **Reasoning quality** | | Is JSON Schema + temperature 0 noticeably better? |
| **Governance & audit** | | Can you explain to an auditor exactly what the agent did and why? |
| **Flexibility & control** | | What could you control here that was impossible in Lab 1? |
| **Maintainability** | | How would you update the playbook? How would you roll back a bad prompt? |

**The trace question:**  
Find one incident where the evaluation failed. What was the root cause —
retrieval, model reasoning, or output format? Write one sentence.

**The moment more control cost you something:**  
Describe one thing that was harder or slower in Foundry than in Studio.

**Foundry IQ vs Studio knowledge:**  
One specific advantage you observed from seeing the retrieved chunks
directly. One scenario where Studio's simpler approach would be sufficient.

> **Keep your Assessment Card.** Lab 3 adds a third column — the combined
> architecture. The Architecture Battle at 16:30 uses all three columns.

---

## Appendix — System prompt reference (Lab 2)

The full system prompt is in `system-prompt-lab2.txt` in the workshop repo.
It is intentionally stricter than the Lab 1 prompt — it demands JSON-only
output with no surrounding text, because the JSON Schema enforcement makes
this reliably achievable. In Lab 1, the same strictness would cause the
agent to occasionally produce broken output when the generative orchestrator
added its own wrapping text.

## Appendix — JSON Schema reference

The full schema is in `json-schema-incident.json`. All enum values are
case-sensitive — `"Critical"` not `"critical"`. The `risk_score` field is
an integer (1–10), not a float and not a string. If the model returns
`"7"` instead of `7`, it fails schema validation and the call returns a
structured error rather than a response.

## Appendix — Workflow endpoint format

Once deployed, the workflow endpoint accepts POST requests in this format:

```
POST https://[your-resource].services.ai.azure.com/api/projects/[project-name]/workflows/[workflow-name]/run
Content-Type: application/json
api-key: [your primary key]

{
  "IncidentText": "your incident report text here"
}
```

The response body is the workflow's final output — the structured JSON
assessment from the agent, with a `Priority` field added by the Condition
node.

Save the full endpoint URL and the API key. You will paste both into
Copilot Studio in Lab 3.

---

*Lab 2 complete. → Proceed to the break, then Lab 3: Wire it Together*
