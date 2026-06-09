# job-tracker-sync

A [Claude](https://claude.com/claude-code) skill that keeps a **Job Application Tracker** Google Sheet up to date automatically by reading your Gmail. Run it on demand, or schedule it (e.g. every Monday at 8am) for a hands-off job-search tracker.

Each run it:

1. Reads your tracker sheet to see what's already there.
2. Scans the last 7 days of Gmail for application confirmations, rejections, interview invites, and other updates.
3. Appends new applications and advances the status of existing ones (forward-only — it never moves a row backward).
4. Writes the changes back into the sheet and reports a short `X new, Y updated, Z flagged` summary.

It never edits your **Notes** column, never deletes rows, and never moves a status backward without flagging it for you to review.

## Prerequisites

You'll need these connectors enabled in Claude:

| Connector | Used for | Access |
|---|---|---|
| **Google Drive** | Reading the current sheet contents | Read-only |
| **Gmail** | Scanning recent emails for application updates | Read-only |
| **Claude in Chrome** (browser extension) | Writing rows/updates back into the sheet | Write (via the open sheet) |

> The Google Drive connector can't edit cells, so writes happen through the Claude in Chrome extension with the sheet open in your browser. If no browser is connected, the run still completes and tells you exactly which rows are pending so you can paste them in or connect Chrome and re-run.

## Setup

1. **Create your tracker sheet** (or use an existing one) in Google Sheets with these columns, in this order:

   `Company | Role | Date Applied | Status | Progress | Last Update | Source | Notes`

2. **Get your sheet's file ID** — it's the long string in the URL between `/d/` and `/edit`:

   `https://docs.google.com/spreadsheets/d/`**`THIS_IS_YOUR_FILE_ID`**`/edit`

3. **Configure the skill** — open [`SKILL.md`](SKILL.md) and replace both occurrences of `<YOUR_SHEET_FILE_ID>` in the **Configuration** section with your file ID.

4. **Install the skill** — place the `job-tracker-sync` folder where Claude looks for skills (e.g. your Claude skills/scheduled-tasks directory), or invoke it directly.

## Running it

- **On demand:** ask Claude to run the `job-tracker-sync` skill.
- **On a schedule:** use Claude's scheduling (e.g. the `/schedule` command or your scheduled-tasks setup) to run it weekly, such as Mondays at 8am.

## Privacy

This repo ships with a **placeholder** sheet ID, not a real one. Your file ID, your sheet contents, and your email never leave your machine/account — the skill is just instructions Claude follows using your own connectors. Keep your real file ID in your local copy only (see `.gitignore`), not in any fork you publish.

## License

MIT — do whatever you like with it.
