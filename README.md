# 💻 MTs Suguja Attendance — Web & Desktop Admin Dashboard

![Flutter](https://img.shields.io/badge/Flutter-3.x-02569B?style=for-the-badge&logo=flutter&logoColor=white)
![Dart](https://img.shields.io/badge/Dart-3.x-0175C2?style=for-the-badge&logo=dart&logoColor=white)
![Supabase](https://img.shields.io/badge/Backend-Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)
![OneSignal](https://img.shields.io/badge/Push-OneSignal_REST-E51B24?style=for-the-badge&logo=onesignal&logoColor=white)
![WhatsApp](https://img.shields.io/badge/WhatsApp-WAHA_Gateway-25D366?style=for-the-badge&logo=whatsapp&logoColor=white)

> **Web & desktop admin portal** built with **Flutter** and **Supabase** for the attendance information system of **MTs Sunan Gunung Jati (Suguja)**. It manages students, teachers, and classes, generates QR student cards as printable PDFs, verifies digital leave letters, sends notifications to parents through **WhatsApp (WAHA)** and **OneSignal push**, and exports reports to Excel, CSV, and PDF.

---

## 📌 Project Overview

This application is the school's administration hub. It brings together master data (students, teachers, classes), real-time daily attendance records, and communication with parents through WhatsApp messages and mobile push notifications.

It is organized into **core services, models, repositories, logic/controllers, and a presentation layer**, and runs both in a **web browser** and as a **desktop app** (Windows, macOS, Linux).

It shares one Supabase database with the mobile companion app **[Mobile-Absensi-Siswa-Mts-Suguja](https://github.com/Aldidwiirawan/Mobile-Absensi-Siswa-Mts-Suguja)** used by teachers, gate scanner officers, and parents.

---

## 🚀 Key Features

### 📊 Interactive Dashboard
- Daily attendance summary: attendance rate and the number of students present, late, on leave, sick, and absent.
- Interactive attendance trend chart (`fl_chart`) with a dynamic Y-axis range and interval.
- Recent attendance activity and system status.

### 🔔 Notifications & Announcements
- **WhatsApp (WAHA)** — direct messages to parents for a single student or a whole class, plus a connection test. Broadcasting to *all* parents via WhatsApp is intentionally skipped to avoid the sender number being banned; OneSignal is used instead.
- **OneSignal push notifications** through the REST API to parents' and teachers' phones.
- Targeting: all parents, per class, or per student.
- Notification test center with attendance alert simulation, message template preview, announcement history, and gateway diagnostics.

### ⚡ Master Data Caching (Stale-While-Revalidate)
- `MasterDataCacheService` reduces latency and database load for classes, teachers, and students.
- The cache is invalidated automatically whenever master data is created, updated, or deleted.

### 🏷️ QR Student Card Generator (`barcode_widget`, `pdf`, `printing`)
- A unique QR code for every student, scanned by the mobile app at the school gate.
- Export and batch-print student cards to PDF with a progress indicator, per student or per class.

### 👨‍🎓 Master Data Management
- **Students** — biodata, NISN, parent contact, and status (*Active, Graduated, Alumni, Transferred / Inactive*).
- **Teachers** — staff data, NIP, subjects, and homeroom assignment.
- **Classes** — class groups, capacity, grade level, and homeroom teacher.
- Debounced instant search.

### ⏱️ Attendance Records
- Daily records with **check-in time**, **check-out time**, status, and an `updated_by` audit trail.
- Filters by date range, status, and class.
- Manual adjustments by the admin.

### 📝 Digital Sick & Leave Letters
- Approve or reject letters submitted by parents from the mobile app.
- Preview the attached photo or doctor's note directly in the dashboard.
- Letter status is synchronized with the daily attendance record.

### 🔒 Authentication & Admin Profile
- Secure login with Supabase Auth.
- Password recovery via an email reset link and a protected update-password page.
- Admin profile: name, email, phone/WhatsApp number, photo, and credentials.

### 📁 Report Export
- **Excel (`.xlsx`) and CSV** export for students, teachers, classes, and attendance recaps.
- **PDF** reports with a print-ready layout.

---

## 🛠️ Technology Stack

| Component | Library / Technology | Purpose |
| :--- | :--- | :--- |
| **Framework** | [Flutter 3.x](https://flutter.dev/) | Web and desktop targets |
| **Language** | [Dart 3.x](https://dart.dev/) | Null-safe, modern OOP |
| **Backend & Database** | `supabase_flutter` | PostgreSQL, Auth, Storage, Realtime |
| **WhatsApp Gateway** | [WAHA](https://waha.devlike.pro/) (WhatsApp HTTP API) via `http` | Direct WhatsApp messages to parents |
| **Push Notifications** | OneSignal REST API | Push to the mobile app |
| **State Management** | `provider` | Centralized app state |
| **Charts** | `fl_chart` | Attendance statistics |
| **QR Code** | `barcode_widget` | Student card QR codes |
| **Documents & Printing** | `pdf`, `printing` | PDF rendering and printing |
| **Spreadsheet Export** | `excel`, `csv`, `archive` | Multi-format report export |
| **Desktop Window** | `window_manager` | Desktop window size and behavior |
| **Configuration** | `flutter_dotenv` | Environment variables from `.env` |
| **File Upload** | `file_picker`, `image_picker` | File and image attachments |

---

## 🌐 System Architecture

```text
   ┌──────────────────────────────┐        ┌──────────────────────────────┐
   │ 💻 Admin Dashboard (this repo) │        │ 📱 Mobile Companion App       │
   │   Flutter Web & Desktop       │        │   Teachers, scanner, parents │
   └───────┬───────────┬──────────┘        └──────────────┬───────────────┘
           │           │                                   │
           │           ▼                                   │
           │   ┌───────────────────┐                       │
           │   │ ☁️ Supabase        │ ◄─────────────────────┘
           │   │ PostgreSQL · Auth │
           │   │ Storage · Realtime│
           │   └───────────────────┘
           │
           ├──────────► 🟢 WAHA WhatsApp Gateway ──► parents' WhatsApp
           └──────────► 🔔 OneSignal REST API ─────► mobile app push notifications
```

---

## 📁 Project Structure

```text
lib/
├── main.dart                               # Environment, Supabase, and window manager initialization
└── web_admin/
    ├── core/
    │   ├── constants.dart                  # Table names, enums, status colors, roles, gateway config checks
    │   ├── exceptions.dart                 # AppException, RepositoryException
    │   ├── theme.dart                      # Material 3 theme
    │   ├── services/
    │   │   ├── error_handler_service.dart  # Centralized error handling & dialogs
    │   │   ├── logging_service.dart        # Logging utilities
    │   │   ├── master_data_cache_service.dart # SWR cache for classes, teachers, students
    │   │   ├── notification_service.dart   # WhatsApp (WAHA) & push notification service
    │   │   └── onesignal_service.dart      # OneSignal REST client
    │   └── utils/debouncer.dart            # Search input debouncer
    │
    ├── data/
    │   ├── models/                         # absensi, guru, kelas, pengumuman, siswa, surat
    │   └── repositories/                   # Supabase repositories with cache invalidation
    │
    ├── logic/                              # Controllers: absensi_controller, kelas_logic, siswa_logic
    │
    └── presentation/
        ├── pages/                          # dashboard, siswa (QR cards), guru, kelas, absensi,
        │                                   # surat, notifikasi_test, profil_admin, password_recovery
        └── widgets/
            ├── dashboard/                  # Stat cards & chart components
            └── notifikasi/                 # Gateway diagnostics, announcement form, simulation, history

test/
├── unit/                                   # Cache service, models, notification enums
└── widget/                                 # Notification dialog widgets
```

---

## ⚙️ Installation & Setup

### Prerequisites
- [Flutter SDK](https://docs.flutter.dev/get-started/install) 3.x (Dart 3.x)
- Google Chrome / Edge for the web build, or the C++ desktop build tools for Windows
- A Supabase project
- Optional: a running [WAHA](https://waha.devlike.pro/) server and a OneSignal app for notifications

### 1. Clone the repository
```bash
git clone https://github.com/Aldidwiirawan/SIM-Dasbor-Absensi-Mts-Suguja.git
cd SIM-Dasbor-Absensi-Mts-Suguja
```

### 2. Install dependencies
```bash
flutter pub get
```

### 3. Create the environment file
```bash
cp .env.example .env
```
Fill in the values described below.

---

## 🔐 Environment Variables

| Variable | Required | Description |
| :--- | :---: | :--- |
| `SUPABASE_URL` | ✅ | Your Supabase project URL |
| `SUPABASE_ANON_KEY` | ✅ | Supabase anon (public) key |
| `WAHA_BASE_URL` | ➖ | Base URL of your WAHA server, e.g. `http://your-waha-server:3000` |
| `WAHA_API_KEY` | ➖ | WAHA API key |
| `WAHA_SESSION` | ➖ | WAHA session name (default: `default`) |
| `ONESIGNAL_APP_ID` | ➖ | OneSignal app ID |
| `ONESIGNAL_REST_API_KEY` | ➖ | OneSignal REST API key |
| `APP_ENVIRONMENT` | ✅ | `development` or `production` |

WhatsApp and push notifications are only active when their variables are set; the admin profile page shows whether each gateway is configured.

> ⚠️ Never commit the `.env` file or real credentials to a public repository.

---

## ▶️ How to Run

**Web (Chrome)**
```bash
flutter run -d chrome
```

**Windows desktop**
```bash
flutter run -d windows
```

**Tests**
```bash
flutter test
```
The test suite covers the master data cache (TTL, freshness, invalidation), model JSON serialization, notification enums, and notification dialog widgets.

---

## 👨‍💻 Author

**SIM Absensi MTs Suguja** is a team project by five students: a Flutter admin dashboard (this repository) and a Flutter mobile app that share one Supabase database.

Team: [@AndhikaBagus115](https://github.com/AndhikaBagus115), [@Sheva1012](https://github.com/Sheva1012), [@adhito18](https://github.com/adhito18), [@bimaibra](https://github.com/bimaibra), and [@Aldidwiirawan](https://github.com/Aldidwiirawan).

**Aldi Dwi Irawan** — built the **Teacher Data (Data Guru)** and **Attendance Data (Data Absensi)** pages of this dashboard.
- GitHub: [@Aldidwiirawan](https://github.com/Aldidwiirawan)
- Portfolio: [aldidwiirawan.vercel.app](https://aldidwiirawan.vercel.app)
- Email: aldidwiirawan2004@gmail.com
