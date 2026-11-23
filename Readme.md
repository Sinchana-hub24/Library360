# Libra360 — Campus Library Circulation & Digital Repository
**Team:** Team 8  
**Course:** Cloud Computing Skill Lab (Project 08 — Campus360)  
**Environment:** Floci GCP Emulator (`127.0.0.1:4588`) & Python FastAPI (`127.0.0.1:8080`)  

---

## 1. Problem Statement
In our college library, physical book checkouts, overdue fine calculations, and digital study PDFs are handled through disconnected manual processes. After our Review 1 feedback (where our initial design resembled a simple ticket-booking), we redesigned **Libra360** to focus on everyday library operations across four main issues:

1. **Manual Fine & Return Errors:** Tracking overdue physical books manually leads to missed due-date checks or accidental duplicate late-fee charges when records are re-processed.
2. **Unverified & Duplicate PDF Uploads:** Digital study resources (lab manuals, notes, and past question papers) lack automated file validation and duplicate checks, resulting in broken files or the same PDF being stored multiple times under different names.
3. **Lack of Proper Role Separation:** Students should only be able to view their own borrowed books, fines, and approved study PDFs—not issue books, mark books as returned, or upload files to the catalog.
4. **No Privacy-Safe Usage Summary:** Librarians need department-wise borrowing and resource counts (CSE, ECE, MECH) without exposing personal student details.

---

## 2. What We Built (Solution Overview)
We built a backend API and single-page web portal that runs locally against the Floci GCP emulator:

- **Librarian-Controlled Physical Circulation:** Librarians (`LIB-ADMIN-01`) issue physical books (`BORROWED` with a due date) and process returns (`RETURNED`, which freezes the accrued fine). Students (`TEST-STUDENT-01`) have read-only access to check their active loans and fines.
- **Idempotent Daily Fine Worker:** A background worker checks overdue physical books and adds a ₹10/day fine using a composite key (`FINE-YYYY-MM-DD-BORROW-ID`). If the sweep runs twice on the same day, it skips already-processed records so a student is never double-charged.
- **PDF Validation & SHA-256 Deduplication:** When a librarian uploads a PDF, the API verifies genuine `%PDF` file headers, enforces a 5 MB limit, computes a `SHA-256` hash to block duplicate files, and stores valid PDFs in Floci Cloud Storage (`gs://libra360-pdfs`).
- **Privacy-Safe Analytics & Network Isolation:** All accounts use synthetic IDs (`TEST-STUDENT-XX`, `LIB-ADMIN-XX`) with zero real student PII. Logs mask user IDs (`***-01`), department summaries run aggregated `GROUP BY` queries, and only the FastAPI gateway (`127.0.0.1:8080`) is exposed to the browser.

---

## 3. User Roles & Permissions
| Role | Synthetic ID | Demo Passcode | Allowed Actions | Restricted Actions (`403 Forbidden`) |
| :--- | :--- | :--- | :--- | :--- |
| **Student** | `TEST-STUDENT-01` to `20` | `pass123` | View own borrowed books, due dates & fines; browse approved digital PDFs | Issue/return books, upload PDFs, run fine sweeps, view department analytics |
| **Librarian (Admin)** | `LIB-ADMIN-01` to `05` | `admin123` | Issue & return physical books, upload PDFs, trigger daily fine sweeps, view department analytics | View raw student IDs in aggregated department reports |

---

## 4. Cloud Services Mapping (Floci Emulator)
| Component | Logical GCP Service | Floci Service | Implementation |
| :--- | :--- | :--- | :--- |
| **API & Web UI** | Cloud Run | `cloudrun` | FastAPI server & HTML portal on `127.0.0.1:8080` (`src/api.py`, `src/index.html`) |
| **PDF Storage** | Cloud Storage | `gcs` | Floci GCS bucket `gs://libra360-pdfs` with SHA-256 deduplication (`src/storage.py`) |
| **Database** | Cloud SQL | `cloudsql` | Relational tables for `students`, `records`, and `processed_fines` |
| **Background Worker** | Cloud Tasks | `cloudtasks` | Idempotent daily overdue fine calculation (`src/worker.py`) |
| **Department Summary** | BigQuery | Mapped to `cloudsql` | `GROUP BY department` SQL query without student IDs |

---

## 5. Folder Structure
- `docs/` — Architecture setup notes and sample JSON schemas (`book-record.json`, `fine-event.json`)
- `src/` — Backend modules (`api.py`, `storage.py`, `worker.py`) and frontend (`index.html`)
- `tests/` — Test cases for invalid inputs, RBAC checks, duplicate PDFs, and idempotent fines
- `requirements.txt` — Python dependencies

---