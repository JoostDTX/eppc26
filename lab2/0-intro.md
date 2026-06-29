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

