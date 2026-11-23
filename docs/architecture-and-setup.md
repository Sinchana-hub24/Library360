# System Flow & Floci Setup Check

## 1. System Flow
Library Counter / Student Upload 
   --> Cloud Run API (Checks inputs)
   --> Firestore (Saves physical borrow records) & Cloud Storage (Stores uploaded PDFs)
   --> Cloud Scheduler & Pub/Sub (Runs daily late-fee checks & validates PDF files)
   --> BigQuery / SQL Summary (Department trends) & Cloud Logging (Safe error logs)

## 2. Floci Service Capability Check
| Cloud Service | Purpose | Lab Status (Native vs. Local Backup) |
<!-- | :--- | :--- | :--- | -->
| **Cloud Run** | Runs our main API | Verify in Floci (or run local Python/Node server) |
| **Firestore** | Stores physical book borrow records | Verify in Floci (or use local JSON/SQLite backup) |
| **Cloud Storage** | Stores uploaded digital PDFs | Verify in Floci (or use local folder backup) |
| **Cloud Scheduler & Pub/Sub** | Triggers daily fine checks & PDF validation | Verify in Floci (or use local timer/event script) |
| **BigQuery** | Department-wise usage summary | Verify in Floci (or use local SQLite query) |
| **Cloud Logging** | Tracks request IDs and errors safely | Verify in Floci (or save structured logs locally) |