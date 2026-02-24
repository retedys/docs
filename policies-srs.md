# DocsOrb Business – Software Specification (Policies Platform)

**Version:** 1.1  
**Date:** 18 Feb 2026  
**Product:** DocsOrb Business – Policy Management Platform (Create · Train · Track · Ask)

---

## 1. Purpose & Scope

DocsOrb helps businesses with Policy Management powered by AI to create, track acknowledgment and be compliant to laws.

This document specifies behaviour for the web application used by:

- Organisation Admins (founders, HR, compliance)
- Policy Editors (legal/HR staff, external legal pros)
- Employees (end-users consuming policies)
- Legal Professionals (template authors, Studio plan)

---

## 2. High-Level Concepts

### 2.1 Policy Object

A **Policy** in DocsOrb is a logical object with:

- **Metadata** (name, category, version, risk level, effective date, tags)
- **Documents** (one or more documents attached via `policy_documents`)
- **Training artefacts** (AI overview, key points; flashcards _planned_)
- **Acknowledgment configuration** (required Y/N, cadence: once / yearly / monthly / custom)
- **Approval workflow** (multi-step approval with configurable approvers)
- **Audience targeting** (groups: departments, teams, locations, custom groups)
- **Stakeholders** (owner, reviewers, contributors)

A policy (e.g. "Corporate Expense Policy") has a single set of metadata but can have **multiple source documents** attached. Each source is one document; combined content is used for training and compliance tracking.

### 2.2 Policy Documents (Attached Sources)

Each **document** attached to a policy is one source. A policy can have **multiple documents**. Each document has a `source_type` from this enum:

| Source Type       | Description                                       |
| ----------------- | ------------------------------------------------- |
| `docsorb_storage` | Created in DocsOrb or uploaded, stored internally |
| `gdrive`          | Linked from Google Drive                          |
| `sharepoint`      | Linked from SharePoint                            |
| `onedrive`        | Linked from OneDrive                              |
| `notion`          | Linked from Notion                                |
| `confluence`      | Linked from Confluence                            |
| `url`             | Public URL (fetched and ingested)                 |
| `other`           | Other source types                                |

When adding or editing a policy, the user can attach documents in these ways:

1. **Create in DocsOrb (from scratch)**
   - Rich text editor (Tiptap-based).
   - Optional AI assist to draft sections based on prompts.
   - Stored as `docsorb_storage` source type.

2. **Create from DocsOrb Template**
   - Choose from legal-vetted board templates with pre-configured policies.
   - Fill company-specific fields; AI can adapt wording.
   - Becomes a `docsorb_storage` source.

3. **Link from Integration**
   - Connect storage (Google Drive, SharePoint, OneDrive, Notion, Confluence).
   - Select one or more existing files; DocsOrb imports content and keeps a link for optional sync.
   - Each selected file is one document with the corresponding source type (e.g. `gdrive`, `sharepoint`).

4. **File Upload**
   - Upload PDF/DOCX/Markdown/TXT.
   - Text is extracted and stored internally (OCR for scanned PDFs via Tesseract.js).
   - Each uploaded file is one document (`docsorb_storage`).

5. **Public URL** _(schema-supported; UI limited)_
   - User provides a publicly accessible URL (e.g. PDF, HTML).
   - DocsOrb fetches and ingests content; stored as `url` source type.

### 2.3 Policy Boards

For employees, policies are grouped into **Policy Boards** – branded collections like "Onboarding Essentials", "Security & Privacy", "HR Basics".  
Boards control layout, branding (logo, colours, cover image), and which employees see which policies.

Board status lifecycle: **Draft → Published → Archived**.

### 2.4 Policy Categories

Every policy belongs to one category:

- `security`
- `privacy`
- `hr`
- `esg` (Environmental, Social, Governance)
- `other`

### 2.5 Risk Levels

Policies can be tagged with a risk level: `low`, `medium`, or `high`.

---

## 3. User Roles & Permissions (Functional Level)

Roles are **granularly configurable** (permissions assigned per user or group). Organisations can define custom permission sets; **presets** provide one-click role assignment.

### 3.1 Default Preset (Out of the Box)

The default preset includes four system role presets:

- **Org Admin**
  - Full control of Create/Train/Track/Ask for their organisation.
  - Manages users, groups, policy boards, integrations, and reminders.
- **Editor**
  - Create/edit policies and boards, see analytics; cannot manage billing/integrations.
- **Employee**
  - See assigned boards, complete training, sign acknowledgments, use Ask.
- **Legal Professional (Studio)**
  - Create and manage templates in a separate workspace; templates can be adopted by customer orgs.

### 3.2 Granular Permissions & Custom Presets

- Permissions are granular (e.g. view policies, edit policies, view analytics, send reminders, view team compliance, manage users, manage integrations).
- Admins can create **custom presets** (e.g. "Manager": read policies + team-level analytics + send reminders to reports) by combining permissions.
- Users are assigned one or more presets (or a custom permission set). No fixed "dedicated" roles; everything is driven by permission sets and presets.

### 3.3 Groups

Users can be organised into **groups** of four types:

- `department`
- `team`
- `location`
- `custom`

Groups are used for audience targeting (which groups should see a policy/board) and for organisational structure.

### 3.4 Object-Level Sharing

Individual policy boards can be shared with specific users at different access levels: `owner`, `admin`, `editor`, `assignee`, `viewer`.

---

## 4. Module Specifications

### 4.1 CREATE – Policy Creation & Management

#### 4.1.1 Requirements

1. **Create Policy – AI from Scratch**
   - Form:
     - Name, category (Security/Privacy/HR/ESG/Other)
     - Description, tags
     - Risk level (Low/Medium/High)
   - Backend sends structured prompt to LLM, returns draft.
   - User edits draft in rich text editor and saves as Draft or Published.

2. **Create Policy – From Board Template**
   - Board template library with pre-configured policy sets.
   - Select a template; it populates a board with multiple policies, categories, and ack cadences pre-filled.
   - User can customise before publishing.

3. **Add Policy – From Integration**
   - Admin connects an integration (Google Drive, SharePoint, OneDrive, Notion, Confluence).
   - File picker lists authorised folders/files.
   - On selection:
     - DocsOrb fetches file text.
     - Stores a copy plus reference to original.
     - Option flag: `auto_sync = true/false`.
   - When `auto_sync` is true and source file changes, DocsOrb re-imports and re-generates training artefacts.

4. **Add Policy – File Upload**
   - Drag-and-drop + browse (Uppy uploader with TUS protocol).
   - Supported: .pdf, .docx, .md, .txt.
   - Perform OCR for scanned PDFs (Tesseract.js).
   - Extracted content displayed for review before creating policy.
   - Multiple files can be attached as separate documents to the same policy.

5. **Add Policy – Public URL** _(schema-supported; UI limited)_
   - User enters a publicly accessible URL (e.g. PDF or HTML).
   - System fetches and ingests content; user can review before attaching as a document.
   - One document per URL; multiple URLs can be added to the same policy.

6. **Policy Metadata & Settings**
   - Fields: name, version, status, category, risk level, owner, description, effective date, tags, primary language.
   - Toggles:
     - Requires Acknowledgment (Y/N)
     - Acknowledgment cadence (once / yearly / monthly / custom)
   - Audience targeting via groups (departments, locations, teams, custom).
   - Stakeholder assignments (owner, reviewers, contributors).
   - Training configuration (default modality settings).
   - A policy can have multiple documents; UI allows attaching additional sources at any time.

7. **Policy Lifecycle**
   - **Draft → In Review → Approved → Archived**
   - The underlying database tracks five states (`draft`, `in_review`, `approved`, `active`, `retired`), but the UI presents a simplified four-state model where `active` maps to "Approved" and `retired` maps to "Archived".
   - Versioning: version string stored per policy (e.g. "1.0", "2.0").

8. **Approval Workflow**
   - Multi-step approval process with configurable approvers.
   - Each step has a status: `pending`, `approved`, `rejected`, `skipped`.
   - Approvers can be assigned from org members.
   - Steps can be reverted to "pending" (moves policy status back if needed).
   - Approval status tracked per policy: `current_approval_status`, `current_approved_at`, `current_approved_by`.

9. **Boards Management**
   - Create/edit boards:
     - Title, description, brand settings (logo, colours, background image).
   - Add policies to boards (each policy belongs to one board via `board_id`).
   - Assign boards to audiences via groups and object-level shares.
   - Board status: Draft → Published → Archived.
   - Board templates for quick setup with pre-defined policy sets.

#### 4.1.2 Non-Functional

- Editor must autosave drafts every 10 seconds.
- Max policy size: 200k characters of text.
- Policy content stored in a format suitable for both search and display (e.g., HTML + structured sections).

---

### 4.2 TRAIN – Employee Training Experience

#### 4.2.1 Requirements

1. **Employee Dashboard**
   - Landing page after login for Employees.
   - Employees see only **published** Policy Boards assigned to them or their group.
   - Displays:
     - Greeting + company branding.
     - Global completion bar (e.g., "6 of 8 required policies completed").
     - List of assigned policy boards.
   - Each board card shows:
     - Board name, description.
     - Policy count and completion ratio.
   - Clicking a board shows its policies as cards:
     - Status icon (Not started / In progress / Completed / Overdue).
     - Due date.
     - Quick action: "Continue" or "Start".

2. **Policy Detail View (for a Single Policy)**

   When an employee clicks a policy card, open the policy detail with these tabs:

   **Editor view tabs:**
   - **Overview** – AI-generated overview, key points, metadata sidebar (owner, category, risk level, version, dates, stakeholders, audience).
   - **Documents** – List of attached policy documents with source type indicators.
   - **Training** – Training materials with status tracking.
   - **Acknowledgments** – Ack campaign management (cadence, deadlines, campaign status).
   - **Approvals** – Multi-step approval workflow with step-by-step progress.
   - **Activity** – Full activity log of all policy events.

   **Employee view tabs:**
   - **Overview** – AI-generated overview and key points.
   - **Documents** – Attached policy documents (read-only).
   - **Training** – Training materials to complete.
   - **Activity** – Activity feed.

3. **Planned: Flashcards** _(Future)_
   - Card stack, one key idea per card.
   - Tap/click to flip front/back; mark as "Understood" or "Review again".

4. **Acknowledgment Flow**
   - If `ack_required = true`:
     - Acknowledgment banner shown with deadline (from active campaign).
     - User clicks to sign; opens signature modal.
     - User types full name as e-signature.
     - System logs: user, policy id + version, timestamp.
     - Status moves to **Acknowledged**.
   - Acknowledgment cadence options:
     - `once` – sign once, done.
     - `yearly` – annual re-acknowledgment.
     - `monthly` – monthly re-acknowledgment.
     - `custom` – configurable schedule.

5. **Acknowledgment Campaigns**
   - Campaigns manage batches of acknowledgment requests.
   - Campaign status: `draft`, `active`, `completed`, `archived`.
   - Each campaign targets a policy version and audience.
   - Individual acknowledgment status: `pending`, `acknowledged`, `expired`.

6. **Branding**
   - Train view uses company logo and primary colour from org/board settings.

#### 4.2.2 Non-Functional

- Optimised for mobile (employees often complete on phone).
- Aim for < 2s load time for Train view on broadband.
- Accessible (WCAG AA: keyboard navigation, contrast, ARIA).

---

### 4.3 TRACK – Analytics, Audit Trails, Reminders

#### 4.3.1 Requirements

1. **Org-Level Dashboard (Admin)**
   - Metrics:
     - Overall completion rate (% of required acknowledgments done).
     - Breakdown by board, policy, department.
     - Trend over time (last 30/90 days).
   - Widgets:
     - "Top Overdue Policies".
     - "Employees at Risk" (most overdue / lowest completion).

2. **Policy Analytics View**
   - For each policy:
     - Total assigned vs trained vs signed.
     - Distribution by department/role.
     - List of employees with statuses:
       - Not started / In progress / Completed (not signed) / Signed / Overdue.
   - Export options: CSV, PDF.

3. **Employee Compliance View (Admin or users with team-view permission)**
   - For one employee:
     - List of all required policies with status + timestamps.
     - Quick links to send targeted reminder.

4. **Audit Trail**
   - Append-only log of:
     - Policy publication/updates.
     - Training completion.
     - Acknowledgment signatures.
     - Reminders sent.
   - Must support filtering by:
     - Policy, employee, date range, event type.
   - Exportable for audits (SOC2, ISO, etc.).
   - Implemented via `policy_activity_log` (per-policy) and `audit_logs` (system-wide).

5. **Reminders System**

   **Automatic Reminders:**
   - On assignment: immediate email + in-app notification.
   - Before deadline:
     - 3 days before
     - 1 day before
     - On due date
   - After deadline:
     - 1 day overdue
     - Weekly until signed (max X times, configurable).

   **Manual Reminders:**
   - From policy or employee views:
     - Multi-select employees → "Send Reminder".
     - Optional custom message.
   - Logged in audit trail.

   **Channels:**
   - Email (Resend integration).
   - Optional Slack/Microsoft Teams notifications (post-MVP).

---

### 4.4 ASK – AI-Powered Q&A _(Planned)_

> The Ask module is designed but not yet implemented in the policy detail view.

#### 4.4.1 Requirements

1. **Policy-Scoped Q&A** _(Planned)_
   - Within policy detail's tabs:
     - Text box: "Ask anything about this policy".
     - Answers generated using retrieval-augmented generation (RAG):
       - Query is embedded.
       - Top-k relevant chunks from that policy pulled.
       - Answer generated strictly from those chunks.
   - Show:
     - Answer text.
     - Expandable "Sources" section listing referenced sections or headings.

2. **Organisation-Scoped Q&A (Global Ask)** _(Planned)_
   - Global Ask entry point (header search bar):
     - Can search across all policies assigned to user.
   - Result = answer + list of policies it referenced.

3. **Safety & Guardrails**
   - If confidence low:
     - "This answer may not be fully reliable. Contact HR/legal."
   - Option to escalate:
     - Button that opens a prefilled email/HR ticket with the question and answer.

4. **Q&A Logging**
   - For each question:
     - User, policy (if scoped), timestamp.
     - Answer & confidence.
   - Admin analytics:
     - Top questions.
     - Policies with most confusion.
     - Questions with low confidence (policy gaps).

---

## 5. Integrations

### 5.1 Storage Integrations (Policy Document Sources)

**Implemented:**

- Google Drive (`gdrive`)
- Microsoft SharePoint (`sharepoint`)
- Microsoft OneDrive (`onedrive`)
- Notion (`notion`)
- Confluence (`confluence`)

**Behaviour:**

- OAuth-based or API key connection per organisation via `storage_providers` table.
- File picker UI for admins when adding a policy document.
- For each linked document, store:
  - Source type (enum value).
  - Source URL / external reference.
  - Last-synced revision/time.
- Optional auto-sync flag:
  - On change notification from provider, re-ingest content and re-generate training artefacts; bump minor version.

### 5.2 Email (Notifications & Invitations)

- **Resend** integration for transactional email (invitations, reminders, notifications).

### 5.3 Payments & Billing

- **Stripe** integration for subscription management.
- Pricing table, billing portal, subscription lifecycle.
- Three tiers: **Founder**, **Startup**, **Scaleup**.

### 5.4 AI

- **Vercel AI SDK** (`ai` + `@ai-sdk/vue`) for LLM integration.
- AI agents (system and user-defined) with configurable capabilities, system prompts, and models.
- AI policy overview and key-point extraction.
- AI usage tracking with token-level logging per company.

### 5.5 HR / Directory Integrations (Assignments & Targeting)

**Roadmap (not strict MVP but designed for):**

- BambooHR, Gusto, Rippling, etc.

**Behaviour:**

- Sync employees and attributes (department, manager, location).
- Auto-assign boards based on rules (e.g., "Onboarding board to all new hires").
- Keep user status in sync (terminated employees disabled).

### 5.6 Compliance Platforms

- Vanta, Drata etc., to export acknowledgment evidence for audits.

**Behaviour:**

- Provide authenticated API endpoints / webhooks to fetch:
  - List of policies and acknowledgement rates.
  - Detailed logs for specific controls.

---

## 6. Key Screens (Behavioural Spec)

### 6.1 Employee – Dashboard

- Header:
  - Company logo, notification bell, profile menu.
- Hero card:
  - "Your compliance status: X/Y required policies completed (Z%)".
- Boards grid:
  - Card per board:
    - Title, description, completion ratio.
- Board detail:
  - List of policy cards with:
    - Name, due date, status, quick "Start/Continue/View" CTA.
- Global search bar:
  - Searches policy titles and across documents.

### 6.2 Admin – Policies / Board List

- Board cards with:
  - Board name, description, status badge, policy count.
- Filters:
  - Board status (Draft/Published/Archived).
- Actions:
  - New Board, New Policy, Use Template.

### 6.3 Admin – Policy Detail

- Header box with:
  - View mode toggle (Editor / Employee preview).
  - Status dropdown (Draft / In Review / Approved / Archived).
  - Edit button, actions menu (duplicate, delete).
- Tabs:
  - Overview, Documents, Training, Acknowledgments, Approvals, Activity.
- Acknowledgment banner (when ack campaign is active for current user).
- Signature modal for e-signing.

### 6.4 Admin – Board Detail

- Header with board name, description, branding.
- Status dropdown (Draft / Published / Archived).
- Policy list with drag-and-drop reordering.
- Assign people modal (add org members/groups as assignees).
- Board settings (logo, colours, background image).

### 6.5 Admin – Team Management

- Three sub-tabs:
  - **Users** – Table of org members with presets, groups, status.
  - **Roles** – Permission presets (system + custom).
  - **Groups** – Department/team/location/custom groups.
- Invite team member modal.
- Change role / manage permissions.

---

## 7. Data Model (High-Level)

_Reflects the actual database schema as of Feb 2026._

### 7.1 Business Schema (`business`)

- **org_members** (auth_user_id, company_id, status, display_name, avatar_url)
- **org_member_permissions** – Direct permission grants per member
- **org_member_presets** – Role preset assignments per member
- **permissions** – Permission definitions (granular capabilities)
- **permission_presets** – Role definitions (system or custom)
- **preset_permissions** – Junction: which permissions belong to which preset
- **groups** (company_id, name, type: department/team/location/custom)
- **group_members** – User-to-group membership
- **object_shares** – Object-level sharing (e.g. share a policy board with a specific user at a given access level)
- **policy_boards** (company_id, name, description, status, branding fields)
- **policies** (board_id, company_id, name, category, status, version, ack_required, ack_cadence, risk_level, owner_id, effective_from, tags, ai_overview, ai_key_points, current_approval_status, framework_links, readiness_flags)
- **policy_documents** (policy_id, source_type, source_url, language) – One-to-many from policy
- **policy_acknowledgments** (campaign_id, org_member_id, policy_version, status, acknowledged_at, signature_data)
- **policy_ack_campaigns** (policy_id, policy_version, status, cadence, deadline, start_at, target_audience)
- **policy_approvals** (policy_id, approver_id, step_order, status, decided_at, assigned_to)
- **policy_stakeholders** (policy_id, org_member_id, role)
- **policy_audience** (policy_id, group_id)
- **policy_trainings** (policy_id, status, modality_config, training_owner_id)
- **policy_activity_log** (policy_id, event_type, event_data, actor_id)
- **policy_board_templates** – Pre-built board templates
- **policy_board_template_policies** – Policies within board templates
- **template_usage_log** – Tracks template adoption

### 7.2 Public Schema (`public`) – Key Tables

- **companies** (owner_id, name, country, website, business_type, industry, branding)
- **company_members** – Legacy company membership (role: owner/editor/viewer)
- **company_invitations** – Pending invitations
- **storage_providers** – External storage configs (GDrive, SharePoint, etc.)
- **ai_agents** (name, capabilities, system_prompt, model, is_global, category)
- **ai_usage_logs** (tokens_used, agent_id, company_id) – AI consumption tracking
- **audit_logs** (action_type, user_id, metadata)
- **user_profiles** – User profile data
- **subscriptions** / **subscription_history** – Stripe subscription state
- **entitlements** – Feature entitlements per subscription

> The public schema also contains legacy tables from a prior document-management version (folders, documents, files, data_rooms, etc.). These are not part of the current policy platform.

### 7.3 Key Enums

| Enum                   | Values                                                                                        |
| ---------------------- | --------------------------------------------------------------------------------------------- |
| `policy_status`        | `draft`, `in_review`, `approved`, `active`, `retired`                                         |
| `board_status`         | `draft`, `published`, `archived`                                                              |
| `policy_category`      | `security`, `privacy`, `hr`, `esg`, `other`                                                   |
| `risk_level`           | `low`, `medium`, `high`                                                                       |
| `ack_cadence`          | `once`, `yearly`, `monthly`, `custom`                                          |
| `ack_status`           | `pending`, `acknowledged`, `expired`                                                          |
| `campaign_status`      | `draft`, `active`, `completed`, `archived`                                                    |
| `approval_status`      | `pending`, `approved`, `rejected`, `skipped`                                                  |
| `training_status`      | `draft`, `in_review`, `approved`, `published`                                                 |
| `org_member_status`    | `active`, `inactive`, `suspended`, `pending_invite`                                           |
| `group_type`           | `department`, `team`, `location`, `custom`                                                    |
| `object_access_type`   | `owner`, `admin`, `editor`, `assignee`, `viewer`                                              |
| `document_source_type` | `docsorb_storage`, `gdrive`, `sharepoint`, `onedrive`, `notion`, `confluence`, `url`, `other` |

---

## 8. Non-Functional Requirements

- **Security**
  - RBAC enforcement on every API (via `user_has_permission` RPC + middleware).
  - Row Level Security (RLS) on all Supabase tables.
  - All data over HTTPS.
  - Passwords hashed via Supabase Auth; OAuth support (Google, etc.).
  - Policy content and acknowledgments must be backed up and retained for audit.

- **Performance**
  - Employee dashboard loads in < 2s on typical corporate connection.
  - AI answer time target < 5s for Ask.
  - Bulk operations (e.g., 1,000 reminders) queued and processed asynchronously.

- **Reliability**
  - Audit logs append-only and tamper-evident.
  - Daily backups with 30-day retention.

- **Multi-Tenancy**
  - Strong separation between organisations via `company_id` on all tables; no cross-tenant data leakage.

---

## 9. MVP vs Later Phases (Engineering Planning Aid)

### MVP (v0) – Implemented

- Default role presets: Org Admin, Editor, Employee (granular permissions; Legal Professional in Studio).
- Create:
  - From scratch editor (Tiptap rich text).
  - From board templates with pre-configured policy sets.
  - File upload (PDF, DOCX, Markdown, TXT with OCR).
  - Link from integrations (Google Drive, SharePoint, OneDrive, Notion, Confluence).
- Train:
  - Policy boards + AI overview + key points.
  - Training materials management.
- Track:
  - Acknowledgment campaigns with configurable cadence.
  - Multi-step approval workflows.
  - Policy activity logging.
  - Audit trail.
- Team:
  - Granular RBAC with permission presets and custom roles.
  - User groups (department, team, location, custom).
  - Object-level sharing for boards.
  - Invitation system.
- Billing:
  - Stripe integration with three tiers (Founder, Startup, Scaleup).
- AI:
  - Configurable AI agents (system + user-defined).
  - AI policy overview and key-point extraction.
  - AI usage tracking.

### Phase 2

- Policy-scoped Ask (RAG-based Q&A within policy detail).
- Flashcards training modality.
- Understanding score metric.
- Configurable reminder schedules.
- Slack/Teams notifications.
- Global Ask (org-scoped Q&A across all assigned policies).

### Phase 3

- HRIS integrations (BambooHR, Gusto, etc.).
- Vanta/Drata compliance exports.
- Policy marketplace, multi-language support, quizzes, mobile app.

---

## 10. Acceptance Criteria (Examples)

- An admin can create a policy from each of the supported sources, assign it to a board, and see at least one employee complete training and sign it.
- An employee can open the Dashboard, see published Policy Boards assigned to them or their group, complete at least one policy, and receive a confirmation email.
- Reminders are automatically sent to employees who have not signed a required policy by the deadline.
- An admin can configure a multi-step approval workflow and see the policy move through Draft → In Review → Approved states.
- Acknowledgment campaigns can be created with different cadences (once, yearly, monthly) and track individual employee sign-off status.
