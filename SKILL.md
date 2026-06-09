---
name: job-tracker-sync
description: Weekly sync of the Job Application Tracker Google Sheet from Gmail (Mondays 8am).
---

You keep Becky's "Job Application Tracker" Google Sheet current from her Gmail. Run this every time, fully self-contained.

TARGET FILE (do not search for a different one):
- Google Sheet, fileId: <YOUR_SHEET_FILE_ID>
- URL: https://docs.google.com/spreadsheets/d/<YOUR_SHEET_FILE_ID>/edit
- Sheet columns, in order: Company | Role | Date Applied | Status | Progress | Last Update | Source | Notes
- Treat each existing (Company + Role) pair as one unique record. Match case-insensitively and tolerate minor wording differences (e.g. "Sr." vs "Senior").

STEP 1 — Read the current table.
Use the Google Drive connector tool read_file_content with the fileId above to read every existing row. Build the list of existing (Company + Role) records and their current Status.

STEP 2 — Search Gmail (last 7 days only).
Use the Gmail connector search_threads, then get_thread (FULL_CONTENT) for any thread that looks job-application-related. Run these queries (all with newer_than:7d):
  a) "application received" OR "thank you for applying" OR "we received your application" OR "your application has been received"
  b) "unfortunately" OR "not moving forward" OR "decided to move forward with other" OR "pursue other" OR "position has been filled" OR "other candidates"
  c) (schedule OR interview OR availability OR "next steps" OR "phone screen" OR assessment) AND (greenhouse OR lever OR workday OR myworkday OR ashby OR ashbyhq OR gem OR recruiting OR ats)
Skip newsletters, job-board digests (LinkedIn/Indeed/etc. "jobs for you"), and automated marketing. Only keep genuine messages about a specific application of hers.

STEP 3 — Classify each relevant email as one of:
  - Applied (confirmation of submission)
  - Rejected
  - Interview (interview scheduled or requested)
  - Other update (assessment, offer, etc.) — record the status text that best fits.

STEP 4 — Reconcile against the sheet:
  - NEW (Company + Role) not already in the table → APPEND a new row. Date Applied = the email date if it's an application confirmation, else leave blank; Status = the classification; Progress = same as Status (e.g. "Applied"); Last Update = today's date (YYYY-MM-DD); Source = the Gmail message link https://mail.google.com/mail/u/0/#all/<threadId>; Notes = blank.
  - ALREADY in the table → update Status and Last Update ONLY. NEVER overwrite Date Applied, Progress (unless empty), Notes, or Source. Notes is Becky's column — never touch it.
  - If an email would move a status BACKWARD (e.g. record is Rejected/Withdrawn/Interview and the email is an "Applied" confirmation, or Interview→Applied), DO NOT change the row. Add it to a "flagged" list to report instead.
  - Status precedence (forward only): Applied < Interview/Other update < Rejected/Withdrawn. Only move forward.

STEP 5 — Write the changes into the sheet.
The Google Drive connector is read-only for cell edits, so edit via the Claude in Chrome browser extension:
  - First check a browser is connected (list_connected_browsers). If none is connected, DO NOT guess — skip writing, and in the summary clearly state that the changes could not be written because no browser was connected, and list the exact rows/updates that are pending so Becky can paste them or connect Chrome.
  - If a browser is connected: open the sheet URL, append new rows at the bottom, and update only the Status and Last Update cells for existing records. Do not delete any rows. Do not reformat existing rows.

STEP 6 — Report a short summary: "X new, Y updated, Z flagged", listing the company/role for each new row, each update (old→new status), and each flagged item with the reason. If writing was skipped, say so explicitly.

Notification: notify Becky on completion with the summary.