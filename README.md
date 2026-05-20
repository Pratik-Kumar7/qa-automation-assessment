# Technical Assessment: Automation & QA System Suite

**Developer:** Pratik Kumar  
**Specialization:** Electronics and Communication Engineering (Final Year)

---

## 🐞 Part 1: Web App QA & Security Audit Report
**Target Deployment Object under Evaluation:** `https://demo.realworld.io`

### Integrated Issue Tracker Registry
Our isolated end-to-end user application matrix tests (Sign-up, Session Auth, Article CRUD pipelines) revealed crucial systemic runtime flows broken across operational scopes:

| # | Title / Summary | Steps to Reproduce | Expected vs Actual | Severity | Suspected Root Cause Axis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | Race Condition: Multi-click Duplicate Submission | 1. Navigate to 'New Article' view.<br>2. Populate mandatory form fields.<br>3. Trigger 'Publish' button multiple times in rapid succession. | **Expected:** UI blocks subsequent inputs; fires single payload.<br><br>**Actual:** UI remains active, processing parallel requests and creating duplicate database rows. | High | Missing `disabled` state toggle or UI component locking on the submit button during active promise resolution. |
| **2** | Unhandled Server Exception on Malformed Email Auth | 1. Access the Sign-up screen.<br>2. Input string lacking standard pattern constraints (e.g., "pratik_kumar").<br>3. Submit the registration form. | **Expected:** Client-side validation blocks execution with a localized formatting alert.<br><br>**Actual:** Form execution bypasses UI check, forcing an unhandled HTTP 500 server exception. | Medium | Client-side Regex input validation hooks are missing or unlinked from the form submission handler. |
| **3** | Stored XSS via Malicious Script Payload Injection | 1. Create a new article draft.<br>2. Inject `<script>alert('XSS')</script>` directly inside the article body field.<br>3. Save, publish, and load the rendered view. | **Expected:** Vulnerable tags should be html-escaped and rendered strictly as static strings.<br><br>**Actual:** The script executes live in the user's DOM context on page render. | Critical | Frontend template uses raw unescaped innerHTML binding blocks instead of passing strings through sanitization filters like DOMPurify. |
| **4** | App Hang / Black-Hole State on Authentication Expiry | 1. Authenticate and log in.<br>2. Clear the active JWT bearer string from browser LocalStorage manually.<br>3. Click any restricted route (e.g., Profile section). | **Expected:** Clean interception of unauthorized traffic with automatic routing back to `/login`.<br><br>**Actual:** Application crashes into a permanent blank screen freeze with no user feedback. | High | Global Axios/Fetch interceptor configuration does not explicitly parse HTTP 401 hooks to force clean path redirects. |
| **5** | Memory Overload via Unbounded Endpoint Pagination | 1. Launch the application landing page.<br>2. Open browser Network Inspector panel.<br>3. Select and examine the global `/articles` GET pipeline. | **Expected:** API fetches structured chunked objects using data cursors.<br><br>**Actual:** The target router dumps the entire unbounded collection array at once, driving high latency. | Medium | Backend controller layer omits explicit default `limit` and `offset` query parameters on database select queries. |

### Deep-Dive Root Cause Engineering Analysis (Issue #1 focus)
The system data duplication anomalies observed on rapid button operations point toward improper async state locking flags management patterns within the UI state interface layers. When an interface request initiates an active HTTP POST, the frontend execution stack fails to isolate and freeze subsequent event handler loops on the physical interaction nodes. Under standard edge latency shifts or quick double-taps, the pipeline maps and shoots parallel runtime server data streams before initial lifecycle responses resolve. 

Because the backend database layer handles these pipelines without unique idempotency verification tokens or transaction-level validation barriers, it creates duplicate record entries for the same action. Fixing this requires hooking an absolute reactive `isSubmitting` tracking boolean state onto the action button element. This ensures the button becomes completely disabled and immutable until the execution state formally settles.

---

## ⚙️ Part 2: Automated n8n Enterprise API Integration Pipeline

### Architectural Blueprint Execution Flow
1. **Cron Schedule Trigger:** Runs an automated scheduler loop evaluated precisely every 1 hour.
2. **Data Acquisition Request:** Queries the standard JSONPlaceholder endpoints to pull active user profile datasets.
3. **Data Transformation (Code Node Engine):** Runs an optimized custom JavaScript map slicing the incoming stream to process just the top 3 users while cleaning unnecessary properties.
4. **Data Enrichment Dynamic Request:** Iterates asynchronously over individual user items via nested parameters loops using targeted expressions parameters: `https://jsonplaceholder.typicode.com/posts?userId={{ $json.userId }}`.
5. **Conditional Logical Branch (IF Node):** Implements dynamic checks filtering content items matching target parameters logic rules (`{{ $json.id }} is greater than 5`).
6. **Output Notifications Engine:** Packages successful evaluations and pushes a customized live incident tracking payload string out to a remote Discord target engine using system-isolated Webhook Credentials.

### Advanced Core Error Resiliency Paradigm
To completely satisfy high-availability production boundaries, the enrichment engine discards default hard failure stops. The critical dependency HTTP endpoints utilize an integrated error parameter setting explicitly mapped to `Continue (using error output)`. If an API request encounters temporary rate-limiting or network timeout faults, the automation loop processes the fault details as an alternative JSON object payload instead of throwing a generic script exception. This design protects the automation pipeline, enabling subsequent active loop indexes to complete their execution runs uninterrupted.
