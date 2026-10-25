Peygir (پیگیر) — Lead Management & Follow-up Workspace
Mission
Transform the existing Persian-language, right-to-left lead and task management application into a polished, reliable, efficient business-development workspace. The current application is provided as `leadmanagement1.202.html` and must be studied before changing anything. This is a product redesign and code-quality initiative, not merely a visual facelift.
Non-negotiable: preserve existing backups and user data
P0 / release blocker: Existing `peygir-backup*.json` files produced by the current HTML application MUST remain importable, without requiring manual editing or losing any information. Keep the current backup workflow usable: choose an existing JSON file, restore the data, and continue saving changes automatically to that same file where the browser permits it. Preserve manual JSON export/import as a fallback.
Current application behavior (verify against source):
State shape: `{ leads: [], tasks: [], settings: { notifEnabled: false, defaultReminder: 60 }, subtaskOrder: [] }`.
Subtasks are embedded within tasks (`task.subtasks`); leads and tasks reference each other via `task.leadId`.
The existing `openExistingBackupFile()` uses `showOpenFilePicker`, parses JSON, validates `leads` and `tasks`, restores state, and holds a writable file handle.
`writeBackupNow()` writes `JSON.stringify(state, null, 2)` through the File System Access API; `persist()` triggers it.
`exportManualBackup()` and `importManualBackup()` support portable JSON backups.
The current app keeps state in memory until backed up; closing the tab without a backup can lose data.
Compatibility requirements
First implement a legacy backup fixture and automated compatibility tests based on the actual source and a safely anonymized sample backup. Do not assume the README is a complete schema.
Import all historical fields and unknown extension fields without silently dropping them. Preserve IDs, relationships, subtasks, subtask ordering, statuses, dates, times, reminders, settings, and product identifiers.
Never overwrite the selected backup on import, on migration, or before explicit user confirmation. Validate and preview first, and keep a rollback copy/snapshot before writing to the original file.
Add versioned, documented migrations internally, while continuing to accept unversioned legacy JSON. Never make legacy compatibility depend on a new `schemaVersion` property.
For maximum backward compatibility, offer a Legacy-compatible JSON export retaining the original root keys and legacy field representations. If a new format is introduced, offer it separately and label it clearly.
Import errors must be actionable and must never destroy currently loaded data. Provide preview (lead/task/subtask counts), validation results, confirmation, and a recoverable restore process.
Queue/debounce writes and serialize them so rapid edits cannot result in out-of-order writes. Surface pending/saved/failed states and the actual last successful save time; never claim success before write completion.
If file access is unsupported or permission is revoked, explain the limitation and allow manual import/export. Do not promise silent background file access across sessions.
Provide local draft protection against accidental tab closure or interrupted writes, while clearly distinguishing local drafts from verified external backups. Avoid silently replacing an existing backup or source of truth.
Add round-trip tests: legacy import → modify → save → reload; and legacy import → legacy export → load in the original HTML app. Include failure cases (invalid JSON, missing arrays, permission denial, interrupted write, duplicate IDs, missing lead references, unknown fields).
Current application: features to retain
The existing single-file HTML/CSS/JavaScript app includes:
Persian RTL UI, Vazirmatn font, Jalali calendar and Persian digits.
Dashboard and sidebar navigation.
Leads with name, company, phone, email, product(s), stage/status, source and notes.
Tasks associated with leads and products, priority, due date/time, reminders, status and description.
Nested subtasks, global subtask view and ordering.
Calendar and browser notifications/reminders.
JSON file backup/restore, automatic writes to a user-selected backup, manual import/export.
Excel import/export for leads, tasks and subtasks using SheetJS.
Existing products: Heevaa (هیوا), Finno (فینو), Dejano (دژانو), and general tasks.
Do not remove working features or change their meaning without documenting and testing an approved replacement.
Product goals and UX redesign
The current UI, UX and business flows need significant improvement. Design for a business-development manager handling multiple organizations, products, follow-ups and meetings. Optimize daily action-taking rather than decoration.
1. Daily command center
Prioritize Today, overdue follow-ups, upcoming meetings, tasks awaiting someone else's response, and neglected leads.
Show meaningful, clickable metrics with clear definitions and filters.
Provide a unified actionable inbox, quick add, search, recent activity and clear empty/loading/error states.
Make overdue and urgent work prominent without overwhelming the user.
2. Lead lifecycle and pipeline
Improve lead creation, qualification, stage transitions, detail pages, editing and archiving.
Provide a clear pipeline/kanban view plus a dense table view, both filterable by product, stage, organization, and activity.
Show lead history and next action. Avoid introducing fields that become mandatory for existing backups.
Make it obvious which leads have no next follow-up and which have stalled.
Preserve original statuses and mappings; any new stages require reversible migration/compatibility handling.
3. Tasks, subtasks and follow-ups
Support fast capture, editing, completion, rescheduling, bulk actions and clear priority/status indicators.
Clarify distinction between tasks, subtasks, follow-up reminders and meetings.
Provide Today / Upcoming / Overdue / Waiting / Completed views, with easy transitions and appropriate confirmation.
Keep tasks tied to leads when relevant, but allow general tasks.
Ensure task and subtask edits immediately update all related views without stale data.
4. Lead detail workspace
One coherent view of contact and organization information, product interest, pipeline stage, associated tasks, upcoming follow-ups, notes and timeline.
Make primary actions discoverable: log interaction, create follow-up, schedule meeting, update stage.
New interaction history features must be optional and compatible with old backups.
5. Calendar and notifications
Preserve Jalali-first workflows and reliable Gregorian storage/conversion.
Provide usable month and agenda views, visible event details and convenient rescheduling.
Distinguish browser notification permission from actual reminder delivery; do not claim guaranteed background notifications when the page is closed.
6. UI/UX quality bar
Modern, restrained enterprise design; Persian RTL, Vazirmatn, accessible contrast and consistent spacing/typography.
Responsive desktop/tablet/mobile layouts; accessible keyboard focus, labels, dialogs and feedback.
Reduce modal overload, unnecessary clicks, visual noise, cramped tables and ambiguous icons.
Use consistent terminology, useful microcopy, inline validation, reversible destructive actions where possible, and informative empty states.
Keep performance smooth for realistically large lead/task datasets.
Technical quality and safety
First audit existing source, state schema, persistence behavior, workflows, edge cases and defects; record findings in `docs/AUDIT.md`.
Refactor incrementally into maintainable modules/components, separating UI, domain logic, persistence, migrations and tests. Choose a stack based on clear tradeoffs; do not rewrite everything blindly.
Maintain a runnable baseline and deliver incremental, reviewable changes.
Add automated unit tests for Jalali conversion, status mapping, relationships, CRUD, filters and backup compatibility; add end-to-end tests for key flows.
Sanitize user-entered strings when rendering HTML; inspect existing `innerHTML` and event-handler patterns for injection risks.
Ensure no accidental data loss during refresh, migration, import, editing or backup failure.
Do not introduce a server, account system, cloud sync, telemetry or third-party data transfer without explicit approval. The current app is local-first.
Preserve Excel import/export functionality and validate malformed input.
Document how to run, test, build and recover data.
Required workflow for Codex
Phase 1 — Analyze (no changes): Read `leadmanagement1.202.html` thoroughly; map existing screens, flows, state schema and persistence. Produce `docs/AUDIT.md`, `docs/UX-REDESIGN.md`, `docs/BACKUP-COMPATIBILITY.md` and a prioritized implementation plan. Explicitly identify data-loss risks.
Phase 2 — Secure compatibility: Add tests and anonymized fixtures for old JSON backup files. Implement robust import/export, safe migrations, draft recovery and truthful save status before major UI changes.
Phase 3 — Improve flows: Implement the daily command center, lead pipeline/detail and task/follow-up workflows incrementally. Keep each phase testable and usable.
Phase 4 — UI polish: Apply the responsive design system and accessibility improvements without losing any feature.
Phase 5 — Validate: Run unit/E2E tests, manually verify legacy backup restoration, export/import, Excel workflows, calendar/reminders, RTL/mobile and failure recovery. Document remaining limitations.
Definition of done
Existing JSON backups import successfully with complete, verified data and preserved relationships.
The user can select their existing backup and continue using it for automatic saves with explicit confirmation and safe rollback.
Legacy-compatible export can be opened by the original app.
All existing functionality remains available and tested.
Main user journeys require fewer steps and present clear next actions.
UI is consistent, responsive, accessible and fully RTL/Jalali-friendly.
Tests pass, critical data-loss/security issues are addressed, and documentation is current.
First instruction to Codex
Read this README and the complete `leadmanagement1.202.html`. Do not start rewriting code yet. First produce a detailed audit of existing functionality, UX friction, technical risks and backup schema, plus a phased redesign proposal. Backup compatibility is P0. After the plan is reviewed, implement in small verified increments.
