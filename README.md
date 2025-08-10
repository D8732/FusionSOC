# FusionSOC

**A hands-on, multi-cloud blue-team lab** that ingests **AWS logs** and **Azure sign-in/VM events** into **Microsoft Sentinel**, correlates identity and workload signals, and triggers detections for **phishing**, **brute force**, and other suspicious activity.

## What this repo contains
- **/images** – screenshots used in documentation
- **/kql** – exact queries used in this lab (copy/paste into Sentinel)
- **/detections** – ready-to-import *Scheduled* rule JSONs for Microsoft Sentinel
- **/docs** – space for notes/diagrams
- **/playbooks** – space for Logic App exports

## Runbook (10,000‑ft)
1. **Install** the **Amazon Web Services** solution in Sentinel (Content Hub).
2. **Create CloudTrail** → logs to **S3** (multi‑region) → configure **SQS**.
3. **Connect** Sentinel’s **AWS S3** data connector (Role ARN + SQS URL).
4. **Enable** **Microsoft Entra ID** + onboard your **Windows VM** with a DCR.
5. **Import detections** from `/detections` or use the queries in `/kql`.
6. **Validate** by generating events (AWS login attempts, IAM user ops, Windows logons, PowerShell).
7. (Optional) **Playbooks** – trigger enrichment/notifications on incidents.

---

## Scenarios

### 1) AWS Console login failures
- **Goal:** detect failed `ConsoleLogin` from `signin.amazonaws.com`.
- **How to use:**
  - Import `detections/aws_console_login_failures.rule.json` (Analytics → **Create** → **Import from JSON**).
  - Or run `/kql/aws_console_login_failures.kql` in Logs to validate.
- **Test:** intentionally fail console login; confirm alert in **Incidents**.    <img width="1321" height="647" alt="Failed logon AWS " src="https://github.com/user-attachments/assets/088937c5-7daa-4c0f-bf99-94f549c70f3a" />


### 2) AWS IAM user created or deleted
- **Goal:** alert on `CreateUser` / `DeleteUser` operations.
- **How to use:** import `aws_iam_user_created_deleted.rule.json` or run `/kql/...created_deleted.kql`.
- **Test:** create/delete a throwaway IAM user; confirm alert and entities.
- **Create Detection rule for whenever a user is created or Deleted in AWS  <img width="1353" height="500" alt="Screenshot 2025-08-09 134031" src="https://github.com/user-attachments/assets/62c3182a-4d8d-40b3-96b4-8e34c1e7dc2f" />
- <img width="1354" height="681" alt="Screenshot 2025-08-09 134604" src="https://github.com/user-attachments/assets/cdea4e05-0486-49ba-8b07-776983720791" />
- <img width="1339" height="652" alt="Screenshot 2025-08-09 140143" src="https://github.com/user-attachments/assets/f52643c5-a8ae-43bd-9b35-2915f83439e6" />




### 3) Windows brute-force (4625)
- **Goal:** detect ≥5 failures within 15 minutes.
- **How to use:** import `windows_bruteforce_4625.rule.json`.
- **Test:** simulate failed RDP/local logons to your lab VM; verify alert.

### 4) PowerShell execution (4688)
- **Goal:** surface PowerShell starts with command line context.
- **How to use:** import `powershell_4688_suspicious.rule.json`.
- **Test:** run `powershell.exe -Command "Get-Process"` and confirm events in `/kql/powershell_4688_events.kql`.

### 5) Phishing enrichment (bonus)
- **Idea:** enrich phishing emails with **VirusTotal**/header analysis and attach evidence to incidents.

---

## Importing the detections (JSON)
1. Portal → **Microsoft Sentinel** → **Analytics** → **Create** → **Import from JSON**.  
2. Paste file contents from `/detections/*.rule.json` and **Create**.  
   - Adjust **Severity**, **Tactics**, and **Entity mappings** as desired.

## Notes
- Keep IAM identities **least‑privileged**.  
- Use **DCR** connector configuration for Windows Security Events to control costs.  
- Normalize custom data with **DCR transforms** if you extend this beyond CloudTrail.

**Author:** Dareis Newton — Winston‑Salem, NC  
**Updated:** 2025-08-10
