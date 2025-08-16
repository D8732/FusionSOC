# FusionSOC

**A hands-on, multi-cloud blue-team lab** that ingests **AWS logs** and **Azure sign-in/VM events** into **Microsoft Sentinel**, correlates identity and workload signals, and triggers detections for **phishing**, **brute force**, and other suspicious activity.

## What this repo contains
- **/images** – screenshots used in documentation
- **/kql** – exact queries used in this lab (copy/paste into Sentinel)
- **/detections** – ready-to-import *Scheduled* rule JSONs for Microsoft Sentinel

## Tool/Services
1. **Install** the **Amazon Web Services** solution in Sentinel (Content Hub).
2. **Create CloudTrail** → logs to **S3** (multi‑region) → configure **SQS**.
3. **Connect** Sentinel’s **AWS S3** data connector (Role ARN + SQS URL).
4. **Enable** **Microsoft Entra ID** + onboard your **Windows VM** with a DCR.
5. **Import detections** from `/detections` or use the queries in `/kql`.
6. **Validate** by generating events (AWS login attempts, IAM user ops, Windows logons, PowerShell).
7.  **Create Detections** – trigger enrichment/notifications on incidents.

---

## Scenarios

### 1) AWS Console login failures
- **Goal:** detect failed `ConsoleLogin` from `signin.amazonaws.com`.
- **How to use:**
  - Import `detections/aws_console_login_failures.rule.json` (Analytics → **Create** → **Import from JSON**).
  - Or run `/kql/aws_console_login_failures.kql` in Logs to validate.
- **Test:** intentionally fail console login; confirm alert in **Incidents**.
- <img width="423" height="566" alt="Screenshot 2025-08-09 120837" src="https://github.com/user-attachments/assets/f35ad327-b7e7-44c4-9e58-a841e6722fe5" />
-  <img width="1321" height="647" alt="Failed logon AWS " src="https://github.com/user-attachments/assets/088937c5-7daa-4c0f-bf99-94f549c70f3a" />
<img width="1339" height="652" alt="AWS Detection rule failed logon" src="https://github.com/user-attachments/assets/20896e0f-a862-472a-97a5-c62ddc4750a7" />



### 2) AWS IAM user created or deleted
- **Goal:** alert on `CreateUser` / `DeleteUser` operations.
- **How to use:** import `aws_iam_user_created_deleted.rule.json` or run `/kql/...created_deleted.kql`.
- **Test:** create/delete a throwaway IAM user; confirm alert and entities,Create Detection rule for whenever a user is created or Deleted in AWS.
- <img width="1353" height="500" alt="Screenshot 2025-08-09 134031" src="https://github.com/user-attachments/assets/62c3182a-4d8d-40b3-96b4-8e34c1e7dc2f" />
<img width="1346" height="645" alt="Delete users AWS" src="https://github.com/user-attachments/assets/06fcffee-79f6-47b9-9257-847c07f03a97" />
- <img width="1354" height="681" alt="Screenshot 2025-08-09 134604" src="https://github.com/user-attachments/assets/cdea4e05-0486-49ba-8b07-776983720791" />
  <img width="1352" height="639" alt="IAM detetction rule" src="https://github.com/user-attachments/assets/787ee32e-26de-4597-8f92-310117d2374e" />





### 3) Windows brute-force (4625)
- **Goal:** detect ≥5 failures within 15 minutes. Then created detection rule
- **How to use:** import `windows_bruteforce_4625.rule.json`.
- **Test:** simulate failed RDP/local logons to your lab VM; verify alert.
- <img width="1329" height="640" alt="Successful and Unsuccessful logon attempts" src="https://github.com/user-attachments/assets/706a8842-191f-4f16-9249-b555e39f3406" />
<img width="1344" height="653" alt="Detection rule for Brute Force" src="https://github.com/user-attachments/assets/5939f4ff-7824-47a0-bf3b-8cf34a115dda" />



### 4) PowerShell execution (4688)
- **Goal:** surface PowerShell starts with command line context.
- **How to use:** import `powershell_4688_suspicious.rule.json`.
- **Test:** run `powershell.exe -Command "Get-Process"` and confirm events in `/kql/powershell_4688_events.kql`.
- Execute Powershell command
- <img width="1095" height="337" alt="Powershell command generate logs" src="https://github.com/user-attachments/assets/7d31a064-f40c-4c11-bc16-6706924ebee8" />
Download file to get events to generate
<img width="1117" height="103" alt="download file in powershell to get commands" src="https://github.com/user-attachments/assets/cfb524c4-15ea-4789-a238-a348252805f6" />
<img width="1356" height="655" alt="Powershell Events Generated" src="https://github.com/user-attachments/assets/966c27d1-50b0-4aea-9c1a-5083e56c2ef8" />
<img width="1342" height="642" alt="Powershell Detection rules" src="https://github.com/user-attachments/assets/d59fae86-e314-4066-8eea-a9777d4e2b77" />





### 5) Phishing Email Analysis
- **Idea:** enrich phishing emails with **VirusTotal**/header analysis and attach evidence to incidents.
- Suspicious email highlighted are the key things to look for in a phishing email to educate end users on.
- 1. the headline using the full email as greeting is always a red flag
  2. Hover mouse over the link should point to a microsoft.com domain, as you see in the bottom left hand corner it points to security-checkup.com, and phishing emails try to get you to act, urgently.  <img width="1360" height="767" alt="Suspicious email" src="https://github.com/user-attachments/assets/b1548447-bdeb-4441-9576-7cbadc34f0d5" />
  Header Email analysis For Analyst to look for
In the the Return path that should be from Microsoft.com instead its from gmail, should raise a red flag
SPf, DKM, and DMARC all state pass should still be looked further into
also the recieved from mail-vk1-f180.google.com should come from microsoft as well
<img width="955" height="737" alt="Email header 1" src="https://github.com/user-attachments/assets/cb852e09-c376-4c97-9553-4ff1c892420f" />
Use Tools like Virus Total to look more into the the Potentially Malicious link to better understand what you are dealing with and next course of Action
<img width="1333" height="636" alt="Virustotal" src="https://github.com/user-attachments/assets/7c32d8a2-538c-4f8b-aa60-037c061e6e4e" />

 

---



## Notes
- Keep IAM identities **least‑privileged**.  
- This links to Controls put in place from Nist 800-53 and Nist CSF https://github.com/D8732/FusionSOC-NIST/blob/main/README.md
- This Links to the Vulnerability Management of the VM used in this Project using Qualys Platform https://github.com/D8732/Azure-VM-Vulnerability-Management-Project-Qualys-Cloud-Agent-Fusion-Soc/blob/main/README.md

**Author:** Dareis Newton — Winston‑Salem, NC  
**Updated:** 2025-08-10
