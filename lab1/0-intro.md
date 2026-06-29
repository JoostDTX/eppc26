# Lab 1 — Build the Incident Coordinator in Copilot Studio

**Duration:** 75 minutes  
**Platform:** Copilot Studio — portal only, no code  
**Azure subscription:** Not required  
**URL:** https://copilotstudio.microsoft.com

---

## What you will build

By the end of this lab you will have a working Incident Coordinator agent that:

- Receives an incident report in plain text
- Classifies it by type, assesses severity and business impact
- Determines the escalation path and recommended action
- Generates a management summary
- Automatically routes the incident via an Agent Flow — posting an adaptive card to Teams and logging a record in Dataverse

You will also run a formal evaluation against the same test dataset used in Lab 2, and take a guided look at governance controls in the Power Platform Admin Center.

---

## Workshop resources — download before you start

From the workshop repository, download these files to your laptop:

| File | Used in |
|------|---------|
| [incident-playbook.pdf](./assets/incident-playbook.pdf) | Part 1 — knowledge source |
| [evaluation-set-10.csv](./assets/evaluation-set-10.jsonl) | Part 4 — evaluation dataset |
| [system-prompt-lab1.txt](./assets/system-prompt-lab1.txt) | Part 1 — agent instructions |
| [adaptive-card-template.json](./assets/adaptive-card-template.json) | Part 2 — Teams card |

