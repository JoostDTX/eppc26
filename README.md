European Power Platform Conference 2026 | Workshop

# Copilot Studio, Microsoft Foundry, or Both? Choosing the Right Architecture for Intelligent Agents

*Workshop level: 300*

👨‍💻 Audience: Developers, Solution Architects, Power Platform or Microsoft practitioners

🗓️ Date: June 29, 2026

📍 Where: Bella Center, Copenhagen, Denmark

🎟️ **SOLD OUT** [Get a ticket](https://espc.tech/conference/eppc-2026/tickets/)!

## Azure sandbox

The below invite code is redeemable at https://aka.ms/JoinEduLab 
UBHMYD

## Workshop prerequisites

> [!CAUTION]
> 🚨 **ACTION REQUIRED**
>
> Complete all prerequisites at least **5 working days before the workshop**.
>
> License approvals, quota requests, and corporate firewall restrictions cannot usually be resolved on the workshop day.

Think of this as charging your batteries before a long drive. Skip it, and you'll spend the first hour watching loading bars while everyone else is already building agents 🙂

# 🚨 Action Required

> [!CAUTION]
> Please complete the following no later than **June 24, 2026**:
>
> - Complete all prerequisites on this page.
> - Fill out the **[participant survey](https://forms.cloud.microsoft/e/FeNWT2eVTZ)**.
>
The survey helps us tailor the workshop to your needs and experience level. Thank you 🙂

---

## 0. Pre-workshop verification - 5 days before

Run through this quickly. If anything fails, there's still time to fix it. Below is detailed information how to configure everything you need for the workshop.

- [ ] `https://copilotstudio.microsoft.com` shows **New Agent** (after you have logged in and clicked on menu **Agent**)
- [ ] `https://ai.azure.com` shows **Create project** (after you have logged in)
- [ ] Power Platform environment with Dataverse exists in [admin.powerplatform.microsoft.com](https://admin.powerplatform.microsoft.com)
- [ ] Azure role is Contributor or Owner and Cognitive Services Contributor + Cognitive Services Usages Reader (Azure Portal → Subscriptions → IAM → View my access)
- [ ] GPT-4.1 quota ≥ 30,000 TPM in Sweden Central or East US 2

---

## 1. Bring your own laptop - and make it personal

**Use a personal laptop, not a corporate one.**

Corporate laptops break workshops. Not sometimes - consistently. The culprits: browser policies that block trial sign-ups, network proxies that intercept OAuth flows, firewall rules that kill Azure WebSocket connections, and MDM policies that reset your settings mid-session.

On a personal device you're in control. When something breaks, you fix it in 30 seconds instead of raising an IT ticket.

**Minimum spec:**
- Any laptop made after 2018 😎
- 8 GB RAM (16 GB preferred - you'll have VS Code + browser + multiple portal tabs open)
- 10 GB free disk space
- Windows 10/11 or macOS 12+

> [!CAUTION]
>**If you genuinely cannot bring a personal laptop**, arrive 30 minutes early and work through the corporate-network issues with the trainer.

---

## 2. Accounts & subscriptions

| What | Why you need it | Get it |
|------|-----------------|--------|
| **Microsoft account** | Sign in to all Microsoft services | [account.microsoft.com](https://account.microsoft.com) - free, 2 min |
| **Microsoft 365 tenant** | Copilot Studio requires Entra ID - personal @outlook.com accounts alone won't work | [1-month Business Basic Trial](https://signup.microsoft.com/get-started/signup?products=91dcd8b1-3b1b-444d-9cdb-0bc0da3eb40d&mproducts=CFQ7TTC0LH18:0002&fmproducts=CFQ7TTC0LH18:0002&culture=en-us&country=us&ali=1) - free 30-day tenant, requires a credit card for identity verification |
| **Copilot Studio trial** | Build and test agents in Lab 1. Trial is full-featured - sign up max 2 days before to avoid expiry during the workshop | [aka.ms/TryCopilotStudio](https://aka.ms/TryCopilotStudio) |
| **Azure subscription** | Labs 2 & 3 (Foundry). Total workshop cost ~$5-30. | [azure.microsoft.com/free](https://azure.microsoft.com/en-us/free/) - $200 credit, credit card for identity only |


**Quick sanity check - do this now:**
- Go to `https://copilotstudio.microsoft.com` → you should see **Create an agent**
- Go to `https://ai.azure.com` → you should see **Create project**

If either fails, fix the account first before proceeding.

---

## 3. Power Platform environment

Copilot Studio stores agents in a Power Platform environment backed by Dataverse. You need one before the workshop.

**Easiest path** - get the free Power Apps Developer Plan, it creates the environment automatically:

👉 [aka.ms/PowerAppsDevPlan](https://aka.ms/PowerAppsDevPlan)

**Already have an M365 account?** Create a Trial environment manually:
1. Go to [admin.powerplatform.microsoft.com](https://admin.powerplatform.microsoft.com) → **Environments → New**
2. Configure environment:
   - Type: **Trial**
   - Dataverse: **Yes**
   - Region: **United States**
   - Get new features early: **Enabled**
3. Save - takes ~5 minutes

**Check your DLP policy (corporate tenants only):**
1. Go to [admin.powerplatform.microsoft.com](https://admin.powerplatform.microsoft.com) → **Policies → Data policies**
2. Find the policy applied to your environment
3. Confirm **HTTP** is in the **Business** group, not **Blocked**

If HTTP is blocked → create a new **Trial environment** for the workshop.

---

## 4. Azure setup & quota - do this 5 days before

This is the most common reason workshops break. Don't skip it.

### 4.1 Choose your region

Use **Sweden Central** or **East US 2**. Both have full GPT-4.1 + Agent Service + File Search support.

### 4.2 Check your Azure role

To fully participate in the hands-on labs, please ensure you have the following permissions assigned before the workshop:

- **Contributor** role on the Azure Subscription (or a dedicated Resource Group)
- **Foundry Owner** role on the Microsoft Foundry resource
- **Cognitive Services Contributor** role to deploy and manage Azure OpenAI models
- **Cognitive Services Usages Reader** role to view quota and usage information

These permissions are required to create and manage Microsoft Foundry projects, deploy models, create AI agents, configure connections, and complete all workshop exercises.

Go to: Azure Portal → Subscriptions → **Access control (IAM) → View my access**

If you only see Reader, switch to a personal subscription or ask your admin.

### 4.3 Register Resource Providers

Ensure the following Azure Resource Providers are registered in your subscription before the workshop:

- Microsoft.Web
- Microsoft.DocumentDB
- Microsoft.Search
- Microsoft.CognitiveServices
- Microsoft.ManagedIdentity
- Microsoft.OperationalInsights
- Microsoft.Insights
- Microsoft.Storage
- Microsoft.KeyVault
- Microsoft.Authorization

To register a Resource Provider, navigate to Azure Portal → Subscriptions → [Your Subscription] → Resource Providers, search for the provider name, and select **Register** if it is not already registered.

### 4.4 Check your GPT-4.1 (and GPT-4.1-mini) quota

The workshop uses **GPT-4.1** (`gpt-4.1`) in **Global Standard** deployment. It's GA, stable, and supported by all Agent Service tools in the labs.

1. Go to [ai.azure.com](https://ai.azure.com) → **Management → Quotas**
2. Filter by your chosen region
3. Find **GPT-4.1 - Global Standard**
4. You need at least **30,000 TPM** available

**If quota is 0 or too low:**
→ Submit a request at [aka.ms/oai/quotaincrease](https://aka.ms/oai/quotaincrease)
→ Request: GPT-4.1 Global Standard · 50,000 TPM · your region
→ Reason: *"Workshop / training lab - one-day usage"*
→ Approval takes 24–48 hours - **submit at least 5 days before the workshop**

---

## Ready?

> ❗ Don't forget to complete the short [survey](https://forms.cloud.microsoft/e/FeNWT2eVTZ) so we can tailor the day to you.

---

## Got questions?

Don't let a quota error or a missing package ruin your workshop day. Reach out **at least 48 hours before** - it's much easier to fix things over a message than live in front of a room full of people.

👉 LinkedIn: https://www.linkedin.com/in/katerinachernevskaya/

Looking forward to seeing you there - quota-approved and ready to build! 🚀

