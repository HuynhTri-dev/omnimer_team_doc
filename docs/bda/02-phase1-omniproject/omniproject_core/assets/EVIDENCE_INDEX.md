# OmniProject Core — Playwright Full-Feature Verification Evidence Index

* **Verification Date:** `2026-11-15`
* **Scope:** All 6 Phases / 24 Functional Requirement Modules (`FR-ORG-001`, `FR-ORG-002`, `FR-PRJ-000` -> `FR-PRJ-022`) + Mobile Responsive NFR (`375x812`)
* **Evidence Directory:** [`assets/`](./) (55 `.png` screenshots: 17 Live Backend Smoke + 38 Full-Feature & Edge-Case E2E)

---

## 1. Executed Test Suites Summary

| Suite | Spec File | Scope | Result | Notes |
| --- | --- | --- | --- | --- |
| **Live Non-Destructive Smoke** | [`omniproject-live-smoke.spec.ts`](../../../../../../frontend_otm/website_otm/tests/e2e/omniproject-live-smoke.spec.ts) | Production `.next` UI -> Render (`https://omnimer-project-core.onrender.com/api/v1`) + Neon DB across all 17 active routes | **1/1 PASSED** (`1.4m`) | `0` page errors, `0` 5xx responses, `404` on `GET /api/v1/subscription` (2x) |
| **Full 24-FR & Edge-Case Evidence** | [`omniproject-full-evidence.spec.ts`](../../../../../../frontend_otm/website_otm/tests/e2e/omniproject-full-evidence.spec.ts) | Contract-matched Playwright API Sandbox across all 6 Phases (24 FRs + `400 OQL_SYNTAX_ERROR` + `410 FORM_CLOSED` + Mobile `375px`) | **6/6 PASSED** (`1.4m`) | 38 full-feature screenshots captured without mutating Neon DB |
| **Phase 6 Dedicated E2E Suites** | `portfolio-dashboard.spec.ts`, `resource-capacity.spec.ts`, `agile-analytics.spec.ts`, `global-search-oql.spec.ts`, `data-migration.spec.ts` | Phase 6 RTM edge cases (WIP, Leave Self-Approval `403`, Burndown backfill, OQL XSS/RBAC, CSV/Jira/Trello AES-256-GCM/ZIP) | **37/37 PASSED** (`2.1m`) | Verified in prior run (before adding the 2 new evidence specs) |
| **Backend TypeScript Build** | `backend_otm/project_core` (`npm run build`) | Full `tsc` compile check | **EXITED 0** | Clean build |

---

## 2. Full 24-FR Visual Evidence Mapping (`P1_*` -> `P6_*`)

### Phase 1 — Multi-Tenant SaaS Foundation, RBAC/SSO & Realtime Engine
| FR ID | Feature Verified | Evidence Screenshot(s) |
| --- | --- | --- |
| `FR-ORG-001` | Tenant Registration, Workspace Billing Quota (`PRO` tier, Invoices) & Audit Trail | [`P1_FR-ORG-001_01_register_tenant.png`](./P1_FR-ORG-001_01_register_tenant.png), [`P1_FR-ORG-001_02_settings_billing_audit.png`](./P1_FR-ORG-001_02_settings_billing_audit.png) |
| `FR-ORG-002` | IAM Invite Members, Role Matrix, User Groups, SAML/OIDC SSO & Accept Invite Onboarding | [`P1_FR-ORG-002_01_iam_invite_groups_sso.png`](./P1_FR-ORG-002_01_iam_invite_groups_sso.png), [`P1_FR-ORG-002_02_accept_invite_onboarding.png`](./P1_FR-ORG-002_02_accept_invite_onboarding.png) |
| `FR-PRJ-000` | Executive Overview Dashboard, Dynamic Island Realtime Radar & OCC Alert | [`P1_FR-PRJ-000_01_overview_realtime_occ_alert.png`](./P1_FR-PRJ-000_01_overview_realtime_occ_alert.png) |

### Phase 2 — Project Lifecycle, Templates, Custom Fields EAV & Subtask Roll-up
| FR ID | Feature Verified | Evidence Screenshot(s) |
| --- | --- | --- |
| `FR-PRJ-018` | Project Hub, Project Templates Tab & Archived Project Read-Only Guard (`unarchive` banner) | [`P2_FR-PRJ-018_01_project_hub_active.png`](./P2_FR-PRJ-018_01_project_hub_active.png), [`P2_FR-PRJ-018_02_project_templates.png`](./P2_FR-PRJ-018_02_project_templates.png), [`P2_FR-PRJ-018_03_archived_readonly_guard.png`](./P2_FR-PRJ-018_03_archived_readonly_guard.png) |
| `FR-PRJ-002` / `FR-PRJ-004` | Custom Fields EAV Schema Manager (`CURRENCY`, `DROPDOWN`, Restore soft-deleted) | [`P2_FR-PRJ-004_01_custom_fields_eav_manager.png`](./P2_FR-PRJ-004_01_custom_fields_eav_manager.png) |
| `FR-PRJ-006` | Hierarchical Subtask Tree & Progress Roll-up (`1/2 = 50%`) inside Task Drawer | [`P2_FR-PRJ-006_01_subtask_tree_progress_rollup.png`](./P2_FR-PRJ-006_01_subtask_tree_progress_rollup.png) |

### Phase 3 — Execution Views, Collaboration, Personal Space & Notification Inbox
| FR ID | Feature Verified | Evidence Screenshot(s) |
| --- | --- | --- |
| `FR-PRJ-001` | Kanban Board (`2/2` WIP indicator, Blocked & Over-Estimate badges) & Table View Custom Columns | [`P3_FR-PRJ-001_01_kanban_board_wip_blocked.png`](./P3_FR-PRJ-001_01_kanban_board_wip_blocked.png), [`P3_FR-PRJ-001_02_table_view_custom_fields.png`](./P3_FR-PRJ-001_02_table_view_custom_fields.png) |
| `FR-PRJ-005` | Markdown Comments, `@Mention` Highlights, Attachment & Activity Stream | [`P3_FR-PRJ-005_01_collaboration_markdown_mentions.png`](./P3_FR-PRJ-005_01_collaboration_markdown_mentions.png) |
| `FR-PRJ-015` | Personal Space `My Tasks` (Overdue/Today/Upcoming) & Global `[Q]` Quick-Add Modal | [`P3_FR-PRJ-015_01_my_tasks_timezone_groups.png`](./P3_FR-PRJ-015_01_my_tasks_timezone_groups.png), [`P3_FR-PRJ-015_02_quick_add_q_shortcut_modal.png`](./P3_FR-PRJ-015_02_quick_add_q_shortcut_modal.png) |
| `FR-PRJ-013` | Notification Inbox, Event Preferences Matrix & Quiet Hours (DND) | [`P3_FR-PRJ-013_01_notifications_inbox_dnd_prefs.png`](./P3_FR-PRJ-013_01_notifications_inbox_dnd_prefs.png) |

### Phase 4 — Agile Scrum, Calendar, Timeline Gantt (CPM), Dependencies & Recurrence
| FR ID | Feature Verified | Evidence Screenshot(s) |
| --- | --- | --- |
| `FR-PRJ-009` | Agile Sprint Planning, Active Sprint Burndown Header & Backlog Drag/Split | [`P4_FR-PRJ-009_01_scrum_sprints_backlog.png`](./P4_FR-PRJ-009_01_scrum_sprints_backlog.png) |
| `FR-PRJ-016` | Interactive Calendar Chips & Unscheduled Tasks Sidebar | [`P4_FR-PRJ-016_01_calendar_chips_unscheduled.png`](./P4_FR-PRJ-016_01_calendar_chips_unscheduled.png) |
| `FR-PRJ-008` / `FR-PRJ-003` | Gantt Timeline with Critical Path (CPM) Highlight & 4-Type Task Dependency Manager (`FS/SS/FF/SF`) | [`P4_FR-PRJ-003_01_gantt_critical_path_highlight.png`](./P4_FR-PRJ-003_01_gantt_critical_path_highlight.png), [`P4_FR-PRJ-003_02_dependency_manager_4_types.png`](./P4_FR-PRJ-003_02_dependency_manager_4_types.png) |
| `FR-PRJ-017` | Recurrence Picker (`RRULE` Frequency, Interval & Regeneration Mode) | [`P4_FR-PRJ-017_01_recurrence_rrule_modal.png`](./P4_FR-PRJ-017_01_recurrence_rrule_modal.png) |

### Phase 5 — Time Tracking, Action Score / Anti-Gaming, Automations & Public Intake
| FR ID | Feature Verified | Evidence Screenshot(s) |
| --- | --- | --- |
| `FR-PRJ-014` | Persistent Floating Timer Widget & Weekly Timesheet Grid | [`P5_FR-PRJ-014_01_floating_timer_and_timesheet.png`](./P5_FR-PRJ-014_01_floating_timer_and_timesheet.png) |
| `FR-PRJ-007` | Action Score `85.0` Smart Sort & Dual-Assignee / Anti-Gaming Date Lock (`MEMBER` view) | [`P5_FR-PRJ-007_01_action_score_antigaming_lock.png`](./P5_FR-PRJ-007_01_action_score_antigaming_lock.png) |
| `FR-PRJ-020` | No-Code Workflow Automations Rules List & Visual `WHEN -> IF -> THEN` Builder | [`P5_FR-PRJ-020_01_nocode_automations_rules.png`](./P5_FR-PRJ-020_01_nocode_automations_rules.png), [`P5_FR-PRJ-020_02_rule_builder_when_if_then.png`](./P5_FR-PRJ-020_02_rule_builder_when_if_then.png) |
| `FR-PRJ-021` | Intake Form Builder, Public Support Portal & `410 FORM_CLOSED` Guard | [`P5_FR-PRJ-021_01_intake_form_builder.png`](./P5_FR-PRJ-021_01_intake_form_builder.png), [`P5_FR-PRJ-021_02_public_intake_portal.png`](./P5_FR-PRJ-021_02_public_intake_portal.png), [`P5_FR-PRJ-021_03_public_intake_closed_410.png`](./P5_FR-PRJ-021_03_public_intake_closed_410.png) |

### Phase 6 — Portfolio Command Center, Capacity, Analytics, Search/OQL, Import/Export & Mobile NFR
| FR ID | Feature Verified | Evidence Screenshot(s) |
| --- | --- | --- |
| `FR-PRJ-010` | Portfolio Health Cards (`ON_TRACK` / `OFF_TRACK` / `NO_DATA`), Program Gantt & Risk Radar | [`P6_FR-PRJ-010_01_portfolio_health_cards.png`](./P6_FR-PRJ-010_01_portfolio_health_cards.png), [`P6_FR-PRJ-010_02_program_gantt_milestones.png`](./P6_FR-PRJ-010_02_program_gantt_milestones.png), [`P6_FR-PRJ-010_03_risk_radar_alerts.png`](./P6_FR-PRJ-010_03_risk_radar_alerts.png) |
| `FR-PRJ-011` | Team Workload Heatmap (`115%` Overload + Confidential Redaction) & Leave Calendar Drawer | [`P6_FR-PRJ-011_01_workload_heatmap_overload.png`](./P6_FR-PRJ-011_01_workload_heatmap_overload.png), [`P6_FR-PRJ-011_02_leave_calendar_drawer.png`](./P6_FR-PRJ-011_02_leave_calendar_drawer.png) |
| `FR-PRJ-012` | Agile Analytics (`Sprint Burndown`, `Velocity Rolling Avg 18.7`, `Flow Metrics`, `Throughput`) | [`P6_FR-PRJ-012_01_agile_analytics_charts.png`](./P6_FR-PRJ-012_01_agile_analytics_charts.png) |
| `FR-PRJ-019` | Global Quick Search (`Cmd+K`, XSS-escaped `<mark>`) & OQL Explorer (`400 OQL_SYNTAX_ERROR` Caret + Saved Filters) | [`P6_FR-PRJ-019_01_cmdk_quick_search_xss_safe.png`](./P6_FR-PRJ-019_01_cmdk_quick_search_xss_safe.png), [`P6_FR-PRJ-019_02_oql_syntax_error_saved_filters.png`](./P6_FR-PRJ-019_02_oql_syntax_error_saved_filters.png) |
| `FR-PRJ-022` | Data Import / Export Migration Hub (CSV/Excel Dry-Run + Trello/Jira + Full ZIP Export) | [`P6_FR-PRJ-022_01_import_export_migration_hub.png`](./P6_FR-PRJ-022_01_import_export_migration_hub.png) |
| `NFR` | Mobile Responsive Verification (`375x812`, `0px` horizontal overflow) | [`P6_NFR_responsive_mobile_375px.png`](./P6_NFR_responsive_mobile_375px.png) |

---

## 3. Live Render + Neon Smoke Evidence (`LIVE_*`)

Captured non-destructively with real login against Render (`https://omnimer-project-core.onrender.com/api/v1`) and Neon PostgreSQL:
* [`LIVE_00_login_screen.png`](./LIVE_00_login_screen.png) -> [`LIVE_01_overview_dashboard.png`](./LIVE_01_overview_dashboard.png) -> [`LIVE_02_projects_hub.png`](./LIVE_02_projects_hub.png)
* Project views: [`LIVE_03_project_kanban.png`](./LIVE_03_project_kanban.png), [`LIVE_04_project_table.png`](./LIVE_04_project_table.png), [`LIVE_05_project_scrum.png`](./LIVE_05_project_scrum.png), [`LIVE_06_project_gantt.png`](./LIVE_06_project_gantt.png), [`LIVE_07_project_calendar.png`](./LIVE_07_project_calendar.png), [`LIVE_08_project_timesheet.png`](./LIVE_08_project_timesheet.png), [`LIVE_09_project_analytics.png`](./LIVE_09_project_analytics.png)
* Workspace views: [`LIVE_10_portfolio.png`](./LIVE_10_portfolio.png), [`LIVE_11_workload.png`](./LIVE_11_workload.png), [`LIVE_12_search_oql.png`](./LIVE_12_search_oql.png), [`LIVE_13_my_tasks.png`](./LIVE_13_my_tasks.png), [`LIVE_14_notifications.png`](./LIVE_14_notifications.png), [`LIVE_15_metrics_contract.png`](./LIVE_15_metrics_contract.png), [`LIVE_16_settings.png`](./LIVE_16_settings.png)

---

## 4. Genuine Gaps & Polish Items Identified

1. **Billing Backend Endpoints Unimplemented (`billing.routes.ts`):**
   * `GET /api/v1/subscription` returns `404` on live Render backend (silently caught by `billing.api.ts`, falling back to hardcoded `FREE` 5 seats / 3 projects quota).
   * `POST /api/v1/billing/portal` is called by `billing.api.ts` but has no route handler in `billing.routes.ts`.
   * `GET /api/v1/billing/invoices/:id/pdf` is marked unimplemented in `billing.routes.ts:42-43`.
2. **Header UI Overlap (`DynamicIsland` vs `AppHeader`):**
   * When WebSocket reconnects or alerts appear (`top-center`), the floating `DynamicIsland` pill overlaps the centered `Tìm kiếm nhanh... ⌘K` search button on desktop (`1440px`) and overlaps the workspace switcher on mobile (`375px`).
3. **Settings Profile Hydration Timing (`LIVE_16_settings.png`):**
   * Direct navigation to `/settings` rendered top-right header name as `User` and empty input placeholders (`fullName`/`timezone`) prior to full profile sync, alongside `FREE` billing fallback.
4. **Documentation Status Sync (`00_DEVELOPMENT_PHASES_ROADMAP.md`):**
   * `omnimer_team_doc` roadmap still marks Phase 6 as in-progress even though Phase 6 backend, frontend UI, and E2E suites (`37/37`) are implemented.
