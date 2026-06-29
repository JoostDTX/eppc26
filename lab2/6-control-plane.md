# Lab 2 — Build the Intelligence Layer in Microsoft Foundry

## Part 6 — Operate: governance walkthrough (10 min)

**Operate** is the governance and administration section of the new Foundry portal. It gives you a cross-project view of your entire Foundry environment — not just the current project. 

> **Navigation:** Click **Operate** in the top navigation bar (Home · Discover · Build · **Operate** · Docs). The left sidebar changes to show Operate-specific items.

---

### Step 1 — Quota

1. Click **Operate** in the top navigation bar.
2. In the left sidebar, click **Quota**.
3. The Quota pane opens showing all model deployments across your Foundry resource with their TPM allocations.
4. Find your `gpt-4.1-mini` deployment. The details pane on the right shows:
   - Current TPM allocation
   - Usage so far
   - Affiliated deployments sharing the same quota pool
5. Toggle **Show all** (top of the list) to see all available models and regions — including ones you have not deployed yet. This is useful for checking capacity before creating a new deployment.

> **The governance point:** Quota is set at the Foundry resource level and applies across all projects within it. An AI operations team controls TPM here — individual project teams cannot exceed their allocation. In Studio, the equivalent is Copilot Credits, but the granularity is very different: there you see consumption by environment, not by model deployment.

---

### Step 2 — Admin

1. In the **Operate** left sidebar, click **Admin**.
2. The Admin pane shows all projects under your Foundry resource with their details: owner, region, connected services, status.
3. Click on your project entry to open its details panel.
4. Observe what is visible at this level: project metadata, connected resources (the AI Search service, Application Insights), and the project owner.

> **The governance point:** The Admin pane provides an enterprise-level lens to oversee and configure multiple projects, user permissions, and linked Azure resources from one place. An AI platform team uses this to audit which teams are running agents, what resources they are connected to, and who owns each project.

---

### Step 3 — Policy and compliance

1. In the **Operate** left sidebar, look for **Compliance**.
2. This section shows the compliance posture across deployments — monitor compliance in real time, surface noncompliant assets, and enable bulk remediation directly from Foundry Control Plane.
3. For per-deployment model settings, navigate via: open the gpt-4.1-mini model on the **Guardrails** tab.
4. Select **Microsoft.DefaultV2** and observe the configurable categories: Hate, Violence, Sexual, Self-harm — each with a severity threshold. This is per-deployment configuration, not tenant-wide. 
5. You can create your Guardrail or Policy.

> **The contrast with Lab 1:** In Studio, content safety is a tenant-level Microsoft-managed setting. You cannot configure thresholds per agent or per deployment. In Foundry, each model deployment can have its own content filter policy.

> **Do not change any settings** — just observe.

---

### Step 4 — Role assignments (RBAC)

1. Still in **Operate** → **Admin**, find your project.
2. Look for an **Access** or **Users** section within the project detail.
3. You should see your account listed as **Foundry Owner**.

> The same Azure RBAC model applies here as for every other Azure resource. A developer gets Foundry User. A team lead gets Foundry Contributor. The platform team holds Foundry Owner. This is not a new concept — it is a familiar control applied to AI.

