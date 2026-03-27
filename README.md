# 📘 LRE Smoke Test – Workflow Overview

This repository contains an automated pipeline that runs a **LoadRunner Enterprise (LRE) smoke test**, collects the HTML report, and sends a **Microsoft Teams notification** with the test results.  
The goal is to provide a **fully automated, repeatable, and observable** test execution process that both technical and non‑technical team members can understand.

---

# 🚀 What This Workflow Does (High‑Level Summary)

When manually triggered, the workflow:

1. **Authenticates** to LoadRunner Enterprise (LRE)
2. **Starts a test run**
3. **Monitors the run** until it finishes
4. **Waits for LRE to generate the HTML report**
5. **Downloads the report** as a ZIP file
6. **Uploads the report** to GitHub Actions as an artifact
7. **Sends a notification** to a Microsoft Teams channel via Power Automate

This gives the team a **single-click smoke test**, with results delivered directly to Teams.

---

# ▶️ How the Workflow Is Triggered

The workflow uses:

```yaml
on:
  workflow_dispatch:
```

This means:

- It **does not run automatically** on push or pull request.
- It is **manually triggered** from the GitHub Actions UI.
- Any team member with appropriate permissions can start a test run.

This ensures the smoke test is run **only when needed**, such as before deployments or during troubleshooting.

---

# 🔐 Variables and Secrets

The workflow uses two types of configuration:

## 1. **Repository Variables** (non‑sensitive)
These store environment‑specific values such as:

| Variable | Purpose |
|---------|---------|
| `LRE_BASE_URL` | Base URL of the LRE server |
| `LRE_DOMAIN` | LRE domain name |
| `LRE_PROJECT` | LRE project name |
| `LRE_TEST_ID` | ID of the test to run |
| `LRE_TEST_INSTANCE_ID` | Instance ID of the test |
| `TENANT_ID` | Tenant identifier for authentication |

These values are safe to store as **GitHub Variables** because they do not contain passwords or tokens.

---

## 2. **Repository Secrets** (sensitive)
These store credentials and private URLs:

| Secret | Purpose |
|--------|---------|
| `LRE_USERNAME` | Username for LRE authentication |
| `LRE_PASSWORD` | Password for LRE authentication |
| `TEAMS_FLOW_WEBHOOK_URL` | Power Automate webhook endpoint |

Secrets are **encrypted** and only available during workflow execution.

---

# 🧠 How the Workflow Works (Step‑by‑Step)

Below is a simplified explanation of what the PowerShell script inside the workflow does.

---

## **1. Authenticate to LRE**
The workflow sends a login request using the username and password stored in GitHub Secrets.

If authentication succeeds, LRE returns a session cookie that is reused for all following API calls.

---

## **2. Start the Test Run**
The workflow sends a POST request to LRE with:

- Test ID  
- Test Instance ID  
- Timeslot duration  
- Post‑run action (“Collate and Analyze”)

LRE responds with a **Run ID**, which uniquely identifies this execution.

---

## **3. Poll the Test Status**
Every 20 seconds, the workflow checks:

- Is the test still running?
- Has it finished?
- Has it failed?

The loop continues until LRE reports either:

- `Finished`
- `Failed`

---

## **4. Wait for the HTML Report**
Even after the test finishes, LRE needs time to generate the HTML report.

The workflow:

- Waits 60 seconds  
- Then checks repeatedly for the `"HTML REPORT"` result type  
- Retries up to 10 times  

Once the report is ready, LRE returns a **Result ID**.

---

## **5. Download the Report**
Using the Result ID, the workflow downloads:

```
LRE-Report.zip
```

This ZIP contains the full HTML analysis report.

---

## **6. Upload the Report as an Artifact**
GitHub Actions stores the ZIP file as a downloadable artifact:

- Easy for QA, developers, and managers to access  
- Preserved with the workflow run history  

---

## **7. Send Notification to Microsoft Teams**
The workflow sends a JSON payload to a Power Automate webhook.

The Teams message includes:

- Run ID  
- Final status (Finished / Failed)  
- Link to the GitHub Actions run  
- Timestamp  
- Message indicating the report is available  

This ensures visibility for the entire team.

---

# 📡 Microsoft Teams Integration (How It Works)

A **Power Automate flow** listens for incoming webhook requests.

When GitHub sends the JSON payload:

1. The flow receives the data  
2. It formats a Teams message  
3. It posts the message into a **private Teams channel** dedicated to test notifications  

This keeps test results centralized and easy to monitor.

---

# 🧩 Why This Workflow Matters

This automation provides:

### ✔ Consistency  
Every test run follows the exact same steps.

### ✔ Transparency  
Teams notifications ensure everyone sees the results.

### ✔ Speed  
No manual login to LRE, no waiting for reports.

### ✔ Traceability  
Artifacts and logs are stored with each run.

### ✔ Security  
Credentials are protected using GitHub Secrets.

---

# 📁 Files in This Repository

| File | Purpose |
|------|---------|
| `.github/workflows/lre-smoke-test.yml` | Main workflow that runs the test |
| `README.md` | Documentation for managers and engineers |

---

# 🎯 Summary

This pipeline automates the entire LRE smoke test process:

- One click to start  
- Automated execution  
- Automated report collection  
- Automated Teams notification  

It reduces manual effort, increases reliability, and provides immediate visibility into test health.
