# U Fix Bah: full app definition

Snapshot of the AppSheet app definition, taken from AppSheet's **App Documentation** page on 12 Nov 2026 (app version 1.000036).

- App: https://www.appsheet.com/start/daef3935-9652-4e4e-be39-eaa13675e06b
- App ID: `daef3935-9652-4e4e-be39-eaa13675e06b`
- Default app folder: `/appsheet/data/JALERTRoadDamageReporter-507114377-26-10-27`
- Totals: 5 tables, 52 columns, 2 slices, 21 views, 17 actions, 0 format rules, 0 workflow rules

## Tables

| Table | Data source | Source / sheet | Updates allowed | Visible |
|---|---|---|---|---|
| ReportsSheet | Google Sheets | JALERT Reports / ReportsSheet | All changes | Always |
| Users | Google Sheets | JALERT Reports / Users | All changes | Always |
| Summary | Google Sheets | JALERT Reports / Summary | Read only | Always |
| Reports | AppSheet database | JALERT Road Damage Reporter-507114377 / Reports | All changes | Always |
| _Per User Settings | Native | _Per User Settings | Updates only | Never |

## Columns

### ReportsSheet (main table)

| Column | Type | Key | Initial value / formula | Notes |
|---|---|---|---|---|
| Report Id | Text | Yes | `UNIQUEID()` | |
| Photo | Image | | | Label column |
| Reported At | DateTime | | `=NOW()` | |
| Location | Address | | `=TEXT(HERE())` | Geocoding on |
| Road Name | Enum (dropdown) | | | Main Gate Road, Library Area, Faculty and Lecture Hall Area, Student Hostel Road; other values allowed. Label column |
| Damage Type | Enum | | | Pothole, Crack, Landslide, Damaged Facility, Unclear, Broken Streetlight, Drainage Problem, Water Supply Issue |
| Severity | Enum | | | Low, Medium, High |
| Ai Description | LongText | | | Filled by AI |
| Upvotes | Number | | | |
| Category | Enum | | | Road Damage, Streetlight, Drainage, Water Supply |
| Status | Enum | | `="New"` | New, Under Review, Assigned, In Progress, Resolved. Editable if `=LOOKUP(USEREMAIL(), "Users", "Email", "Role")="Admin"` |
| Assigned Agency | Enum | | | JKR Sabah, SESB, JPS Sabah, "Jabatan Air SabahUMS Security", UMS Facilities, UMS Maintenance. Editable only by Admin (same rule as Status) |
| Impact Assessment | LongText | | | |
| Resolved At | DateTime | | | |
| Resolution Notes | LongText | | | |
| Priority Score | Number (virtual, read-only) | | see below | |

Priority Score app formula:

```
=IF([Status]="Resolved", 0,
  SWITCH([Severity], "High", 30, "Medium", 20, "Low", 10, 0)
  + SWITCH([Damage Type], "Landslide", 15, "Pothole", 10, "Drainage Problem", 8,
      "Water Supply Issue", 8, "Broken Streetlight", 8, "Damaged Facility", 5, "Crack", 3, 0)
  + MIN(LIST(IF(ISBLANK([Upvotes]), 0, [Upvotes]), 20))
  + IF(ISBLANK([Road Name]), 0,
      MIN(LIST(MAX(LIST(COUNT(SELECT(ReportsSheet[Report Id],
        AND([Road Name] = [_THISROW].[Road Name], [Status] <> "Resolved"))) - 1, 0)) * 3, 15))))
```

### Users

| Column | Type | Key | Notes |
|---|---|---|---|
| Email | Email | Yes | Marked as sensitive data (PII) |
| Name | Name | | Label column |
| Role | Enum | | Admin, Agency, Student |
| Agency | Text | | |

### Summary

| Column | Type | Key | Notes |
|---|---|---|---|
| Summary Id | Text | Yes | `UNIQUEID()` |
| Generated At | DateTime | | `NOW()`, label column |
| Summary | LongText | | AI-generated summary |

### Reports (original version 1 table)

| Column | Type | Notes |
|---|---|---|
| Row ID | Text (key, hidden) | `UNIQUEID("PackedUUID")` |
| Report Id | Text (hidden) | `UNIQUEID("UUID")` |
| Photo | Image | Label column |
| ed At | DateTime | Column name appears truncated (probably meant "Reported At") |
| Location | Address | |
| Road Name | Name | |
| Damage Type | Enum | Pothole, Crack, Landslide, Damaged Facility, Unclear |
| Severity | Enum | Low, Medium, High |
| Ai Description | LongText | |
| Upvotes | Text | |

### _Per User Settings

System table: `_EMAIL` (`USEREMAIL()`), `_NAME` (`USERNAME()`), `_LOCATION`, `_THISUSER` (key), plus unused template options (Option 1–9, Country Option, Language Option).

## Slices

| Slice | Source table | Row filter |
|---|---|---|
| Sabah Reports | ReportsSheet | `=CONTAINS([Location], "Sabah")` |
| Open High Priority | ReportsSheet | `=AND([Status]<>"Resolved", [Severity]="High")` |

## Views

| View | Type | Position | Configuration |
|---|---|---|---|
| Case Tracking | Deck | Center | Header: Damage Type / Reported At, summary: Severity. Grouped by Status, sorted by Reported At (newest first) |
| Priority Ranking | Deck | Left | Image: Photo. Header: Report Id / Status, summary: Priority Score. Sorted by Priority Score (highest first) |
| Map | Map | Right | Map column: Location |
| Dashboard | Dashboard | Right | Tabs: Priority Ranking, By Category, By Severity, By Status, Open High Priority |
| AI Summary | Detail | Menu | Shows Generated At and Summary, newest first |
| By Category | Donut chart | Menu | Column: Category, percent labels |
| By Severity | Donut chart | Menu | Column: Severity, percent labels |
| By Status | Donut chart | Menu | Column: Status, percent labels |
| Open High Priority | Card | Menu | Open High Priority slice |
| ReportsSheet deck | Deck | Menu | Image: Photo |

System-generated detail and form views (position "ref"): Open High Priority_Detail / _Form, Reports_Detail / _Form, ReportsSheet_Detail / _Form, Sabah Reports_Detail / _Form, Summary_Detail, Users_Detail / _Form.

## Actions

| Table | Action | Type | Condition |
|---|---|---|---|
| ReportsSheet | Add | Add record | |
| ReportsSheet | Edit | Edit record | |
| ReportsSheet | Delete | Delete record | `=LOOKUP(USEREMAIL(), "Users", "Email", "Role") = "Admin"` |
| ReportsSheet | View Map (Location) | Navigate app | `NOT(ISBLANK([Location]))` |
| ReportsSheet | Upvote | Set column value | `true` |
| ReportsSheet | Chart Data for By Category | Navigate app | `=CONTEXT("View") = "By Category"` |
| ReportsSheet | Chart Data for By Severity | Navigate app | `=CONTEXT("View") = "By Severity"` |
| ReportsSheet | Chart Data for By Status | Navigate app | `=CONTEXT("View") = "By Status"` |
| Reports | Add / Edit / Delete | Record actions | |
| Reports | View Map (Location) | Navigate app | `NOT(ISBLANK([Location]))` |
| Users | Add / Edit / Delete | Record actions | |
| Users | Compose Email (Email) | Email | `NOT(ISBLANK([Email]))` |

## Updating this file

AppSheet has no Git sync. To refresh this snapshot: in the AppSheet editor open **Settings → Information → App documentation**, then copy the changes here (or use *Print or Save as PDF* and commit the PDF).
