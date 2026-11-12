# U Fix Bah

UMS community amenities reporting and prioritisation app for roads, streetlights, drainage and water supply, built with [Google AppSheet](https://about.appsheet.com/).

**[Open the app](https://www.appsheet.com/start/daef3935-9652-4e4e-be39-eaa13675e06b)** · [Full app definition](docs/app-definition.md)

Students and drivers report a problem with a photo. The app records the time and GPS location, AI fills in the damage type, severity and a short description, and every open report gets a **Priority Score** so the most urgent issues rise to the top.

> This is a no-code AppSheet app. AppSheet does not export apps as source code, so this repository holds the app's documented definition (tables, columns, formulas, slices and views) rather than runnable code. The live app and its data live in AppSheet and Google Sheets.

## Features

- **Photo reporting** with automatic timestamp (`NOW()`) and location (`HERE()`)
- **AI analysis** of each photo: damage type, severity and a one-sentence description
- **Priority ranking** of open reports using a scoring formula (see below)
- **Case tracking** grouped by status: New → Under Review → Assigned → In Progress → Resolved
- **Map** of all reports around Kota Kinabalu / UMS
- **Dashboard** with charts by category, severity and status, plus an "Open High Priority" list
- **Role-based editing**: only Admins can change Status and Assigned Agency

## Data

| Table | Source | Purpose |
|---|---|---|
| ReportsSheet | Google Sheet "JALERT Reports" | Main reports table |
| Users | Google Sheet "JALERT Reports" | Email, name, role (Admin / Agency / Student), agency |
| Summary | Google Sheet "JALERT Reports" | AI-generated summaries (read-only) |
| Reports | AppSheet table | Original reports table from the first version |

### ReportsSheet columns

| Column | Type | Notes |
|---|---|---|
| Report Id | Text (key) | `UNIQUEID()` |
| Photo | Image | |
| Reported At | DateTime | `=NOW()` |
| Location | Address | `=TEXT(HERE())` |
| Road Name | Enum | Main Gate Road, Library Area, Faculty and Lecture Hall Area, Student Hostel Road (others allowed) |
| Damage Type | Enum | Pothole, Crack, Landslide, Damaged Facility, Unclear, Broken Streetlight, Drainage Problem, Water Supply Issue |
| Severity | Enum | Low, Medium, High |
| Ai Description | LongText | Filled by AI |
| Upvotes | Number | |
| Category | Enum | Road Damage, Streetlight, Drainage, Water Supply |
| Status | Enum | New, Under Review, Assigned, In Progress, Resolved. Admin-only edit |
| Assigned Agency | Enum | JKR Sabah, SESB, JPS Sabah, Jabatan Air Sabah, UMS Security, UMS Facilities, UMS Maintenance. Admin-only edit |
| Impact Assessment | LongText | |
| Resolved At | DateTime | |
| Resolution Notes | LongText | |
| Priority Score | Number (virtual) | See formula below |

## Priority Score

Resolved reports score 0. Otherwise the score is the sum of:

| Factor | Points |
|---|---|
| Severity | High 30, Medium 20, Low 10 |
| Damage type | Landslide 15, Pothole 10, Drainage / Water Supply / Streetlight 8, Damaged Facility 5, Crack 3 |
| Upvotes | 1 per upvote, max 20 |
| Other open reports on the same road | 3 each, max 15 |

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

## Slices

| Slice | Filter |
|---|---|
| Sabah Reports | `=CONTAINS([Location], "Sabah")` |
| Open High Priority | `=AND([Status]<>"Resolved", [Severity]="High")` |

## Views

| View | Type | Position | Details |
|---|---|---|---|
| Priority Ranking | Deck | Bottom tab | Sorted by Priority Score, highest first |
| Case Tracking | Deck | Bottom tab | Grouped by Status, newest first |
| Map | Map | Bottom tab | Pins by Location |
| Dashboard | Dashboard | Bottom tab | Priority Ranking, By Category, By Severity, By Status, Open High Priority |
| By Category / By Severity / By Status | Donut chart | Menu | |
| Open High Priority | Card | Menu | |
| AI Summary | Detail | Menu | Latest AI summary |

## App info

- Platform: Google AppSheet
- Version: 1.000036
- Data: Google Sheets
