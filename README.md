# 🏥 E-SIR T — Hospital Management System (Frontend Contribution)

> **Note:** This is a portfolio documentation repository. The source code is proprietary and owned by Azziptech. This README documents my frontend engineering contributions to the production system live at [esirt.co.in](https://esirt.co.in).

---

## 📌 Project Overview

**E-SIR T** (Electronic System for Interns, Residents & Trainees) is a production-grade, full-stack Hospital Education & Management System serving medical institutions. The platform handles the complete lifecycle of hospital operations — from student onboarding and patient case tracking to equipment inventory management.

| Detail | Info |
|---|---|
| **My Role** | Frontend Developer (Student Portal + Admin Panel) |
| **Stack** | Next.js 15, React 19, Tailwind CSS v4, Firebase, Axios, Socket.IO |
| **Type** | Production Web Application |
| **Live URL** | [https://esirt.co.in](https://esirt.co.in) |
| **Users** | Students, Interns, Residents, Hospital Staff, Admins |

---

## ⚙️ Tech Stack

```
Next.js 15.2.3 (Pages Router)   React 19.0.0          Tailwind CSS v4
Formik + Yup                    Axios 1.8.4            Firebase 11.5.0
Socket.IO Client 4.8.1          Framer Motion 12.5.0   AOS 2.3.4
jsPDF + jsPDF-AutoTable         ExcelJS 4.4.0          Chart.js 4.4.4
Plyr 3.7.8                      dom-to-image           Select2 + jQuery
React Quill                     Lucide React           React Toastify
```

---

## 🧩 My Contributions

I worked on the **student-facing frontend portal** (`front-esirt`) and contributed to specific modules in the **admin panel** (`admin-esirt`). My work focused on the **Management & Operations** feature set.

### ✅ Features I Built

#### 🩺 Complaint Management System
- Full complaint lifecycle UI — raise, view, edit, and track status
- Three-modal pattern: View details / Edit complaint / Track response
- **QR Code-based complaint entry** — patients/staff scan a QR, the `trackingId` in the URL pre-fills the complaint form automatically (zero manual steps)
- Department, Doctor, and Ward reusable dropdown components
- Formik + Yup form validation with inline error feedback
- Body scroll lock pattern on modal open/close
- Sortable columns, search/filter, and custom pagination

#### 👤 Patient Reference Module
- Complete CRUD interface for managing patient case references for interns and residents
- View / Edit / Status tracking via multiple modals
- Reusable Department, Doctor, Ward dropdowns shared with complaint module
- AOS-animated list entries, ClipLoader for async states, React Toastify notifications

#### 🔧 Equipment Inventory & Calibration
- Equipment full lifecycle management — purchase details to stock ledger
- Equipment inventory tracking with Excel export (ExcelJS)
- Calibration record management with calibration detail pages
- Warranty tracking and equipment status views

#### 🔐 Authentication Flow
- Login page with Formik + Yup dual validation (accepts email format OR 10-digit mobile number)
- Base64 password encoding before API transmission
- **Date-based session expiry** — Header compares Base64-encoded login date with today's date on every load; auto-logout at midnight without a server call
- Post-login redirect support including `trackingId` passthrough for QR flows

#### 🧭 Global Architecture
- `_app.js` auth guard — whitelist-based public route protection, redirects unauthenticated users
- Animated splash screen on boot (Hospital + Heart icons, gradient loading bar)
- Axios singleton (`api.js`) with hostname-based environment switching (local ↔ production)
- Base64 ID encryption/decryption (`enc_dec_ids.js`) to prevent raw DB ID exposure in URLs
- Auth header factory functions (`headers.js`) — JSON and multipart variants for all API calls

#### 🧱 Reusable Component Library
Built shared components consumed across 10+ pages:
- `Departmentdropdown.js`, `Warddropdown.js`, `Doctordropdown.js` — API-fetched searchable selects
- `DataTable.js` — reusable sortable, searchable, paginated data table
- `LoaderWithMsg.js` — spinner with custom message
- `FileContent.js` — file type detector that renders images, PDFs, videos, or download links appropriately

---

## 🏗️ Architecture Highlights

### Hierarchical Dropdown Pattern
Course → Subject → Module cascades used across multiple pages. Each level fetches the next only after the previous is selected — reducing unnecessary API calls.

### Form-First Development
Every data entry feature starts with a Formik schema and Yup validation definition. UI is layered on top. This ensures consistent error handling and field state management across all forms.

### Session State Strategy
```
localStorage    →  Long-term auth (token, username, role, loginDate)
sessionStorage  →  Short-term filter state (selected course/subject)
React state     →  Component-level ephemeral data (modal open, loading flags)
```

### Modal-Based Detail Views
Detail modals are preferred over page redirects for record viewing and editing. This preserves list state, avoids full page reloads, and keeps the UX fast.

### QR Code Workflow
```
User scans QR on patient form
    ↓
URL: /?trackingId=XYZ
    ↓
index.js detects trackingId
    ↓
Logged in?  →  /add-complaint-raised?trackingId=XYZ
Not logged in?  →  /login?redirect=add-complaint-raised&trackingId=XYZ
                    (trackingId preserved through login, redirected after auth)
```

---

## 📁 Project Structure (Frontend — My Scope)

```
front-esirt/
├── src/
│   ├── pages/
│   │   ├── _app.js                  ← Auth guard + splash screen + layout
│   │   ├── index.js                 ← QR-aware smart entry point
│   │   ├── login.js                 ← Formik login with dual validation
│   │   ├── dashboard.js             ← Student dashboard
│   │   ├── complaint.js             ← Complaint list (view/edit/status)
│   │   ├── add-complaint.js         ← Raise new complaint
│   │   ├── add-complaint-raised.js  ← QR-based complaint entry
│   │   ├── complaint-raised.js      ← Track QR complaints
│   │   ├── patient-reference.js     ← Patient case reference list
│   │   ├── add-patient-reference.js ← Add patient reference
│   │   └── components/
│   │       ├── Header2.js           ← Navbar with date-expiry session logic
│   │       ├── Departmentdropdown.js
│   │       ├── Warddropdown.js
│   │       └── Doctordropdown.js
│   └── utils/
│       ├── api.js                   ← Axios singleton with env switching
│       ├── firebase.js              ← Firebase Auth initialization
│       ├── enc_dec_ids.js           ← Base64 ID encryption
│       └── headers.js              ← Auth header factories
```

---

## 🚀 Key Engineering Decisions

**Why Base64 ID encryption?**  
Raw database IDs in URLs are a security risk — they expose the data model and allow enumeration attacks. Every ID passed in a URL or API call is Base64 encoded/decoded via a shared utility.

**Why hostname-based API switching instead of `.env`?**  
The Axios singleton automatically detects `window.location.hostname` to switch between local dev and production API endpoints. This removes the need to change environment files between deploys.

**Why a date-based session expiry instead of token TTL?**  
The app required sessions to expire at midnight regardless of when the user logged in. Storing the login date as a Base64 string in `localStorage` and comparing it on every header render achieves this entirely client-side with no extra server round-trips.

---

## 📊 Project Scale

| Metric | Count |
|---|---|
| Total frontend pages | 55+ |
| Total admin panel pages | 100+ |
| Total libraries used | 50+ |
| Reusable components built | 15+ |
| API integrations | 30+ endpoints |

---

## 🔒 Note on Code Access

This project is proprietary — the source code belongs to Azziptech and cannot be shared publicly. This repository exists to document my technical contributions, architectural decisions, and the engineering patterns I applied while working on the project.

For questions about my work or approach, feel free to reach out.

---

*Built with Next.js 15 · React 19 · Tailwind CSS v4 · Firebase · Socket.IO*
