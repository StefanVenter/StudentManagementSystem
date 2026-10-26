# Student Management System (SMS)

## Project
- **Project title:** Student Management System – a secure desktop application for managing departments, courses and student enrolments
- **Team:** Stefan Venter, Marcus Janecke, Janco Geldenheis (CTU Training Solutions – Occupational Certificate: Software Engineer)
- **Supervisor:** Mr. Lusukama Selemani
- **Problem:** Small training institutions and departmental registry offices often keep student, course and department records in spreadsheets and paper files. Records are duplicated, students end up linked to courses that no longer exist, e-mail addresses are captured twice, and simple questions ("how many students are in Software Engineering?") take manual counting. There is no access control over who may change or delete records.
- **Proposed solution:** A Windows desktop application backed by a normalised (3NF) Microsoft SQL Server database that enforces referential integrity (Department → Course → Student), validates every input, restricts deletions to administrators, and produces enrolment reports that can be exported to Excel (CSV).

## Technology Stack
| Layer | Technology |
|---|---|
| Presentation | C# 12, .NET 8, Windows Forms (`StduentManagementSystem`) |
| Business logic | .NET 8 class library – validation, services, security (`StduentManagementSystem.Core`) |
| Data access | ADO.NET with `Microsoft.Data.SqlClient` 5.2, parameterised SQL, repository pattern |
| Database | Microsoft SQL Server 2019+/Express, managed with SSMS |
| Security | PBKDF2-SHA256 salted password hashing, role-based access (Admin / Staff), log-in lock-out |
| Testing | MSTest 3.6 unit tests (`StduentManagementSystem.Tests`) |
| Tools | Visual Studio 2022 (17.13+) / 2026, Git & GitHub |

## Solution Structure
```
05_Source_Code/
├── StduentManagementSystem.slnx            Solution (open this in Visual Studio)
├── database/
│   ├── 01_create_schema.sql                Database, tables, constraints, indexes, views
│   ├── 02_seed_data.sql                    Departments, courses, 16 students, 2 user accounts
│   └── 03_crud_and_report_queries.sql      CRUD, JOIN, aggregate and constraint-test queries
├── StduentManagementSystem/                Windows Forms UI (start-up project)
│   ├── Program.cs                          Start-up: log-in → main window
│   ├── AppServices.cs                      Composition root (wires repositories → services)
│   ├── AppSettings.cs                      Reads the connection string
│   ├── appsettings.json                    Connection string (no secrets)
│   └── Forms/                              LoginForm, MainForm, StudentsForm, CoursesForm,
│                                           DepartmentsForm, ReportsForm, CrudFormBase, Ui
├── StduentManagementSystem.Core/           Business + data layer (no UI code)
│   ├── Models/                             Department, Course, Student, AppUser, report models
│   ├── Validation/                         Business rules (one place, reused by UI and tests)
│   ├── Services/                           Department/Course/Student/Auth/Report services
│   ├── Data/                               Repositories + SQL error translator
│   ├── Security/                           PasswordHasher (PBKDF2)
│   ├── Exceptions/                         Friendly, typed exceptions
│   └── Export/                             CSV exporter
└── StduentManagementSystem.Tests/          48 MSTest test methods (96 cases incl. data rows)
```

## Setup
1. **Clone the repository** (or extract this folder).
2. **Install prerequisites:** Visual Studio 2022 17.13+ (or VS 2026) with the *.NET desktop development* workload, and SQL Server Express + SSMS.
3. **Create the database:** open SSMS, connect to your server (e.g. `localhost\SQLEXPRESS`) and run, in order:
   1. `database/01_create_schema.sql`
   2. `database/02_seed_data.sql`
4. **Configure the connection:** edit `StduentManagementSystem/appsettings.json` if your server name differs, or set the `SMS_CONNECTION_STRING` environment variable (see `.env.example`). Never commit real passwords.
5. **Run the application:** open `StduentManagementSystem.slnx`, right-click **StduentManagementSystem** → *Set as Startup Project*, press **F5**. NuGet packages restore automatically.
6. **Run the tests:** *Test → Run All Tests* (or `dotnet test` in this folder). The tests use in-memory fakes and do not need SQL Server.

## Demonstration Accounts
| Username | Password | Role | Can do |
|---|---|---|---|
| `admin` | `Admin@123` | Admin | Everything, including deleting records |
| `staff` | `Staff@123` | Staff | Add, edit, search, report, export – **cannot delete** |

These are demo-only credentials stored as PBKDF2 hashes (never plain text).

## Demonstration Sequence (assessment)
1. Log in with a wrong password twice → generic error with attempts remaining (no user enumeration).
2. Log in as `staff` → dashboard shows totals and the department overview; note the **Delete** button is disabled.
3. **Students** → filter by *Information Technology*, search "smi", add a new student with an invalid e-mail and a 9-digit phone → all validation messages appear together.
4. Correct the data and save → the student appears in the list; try saving a second student with the same e-mail → duplicate message.
5. Log out, log in as `admin` → **Departments** → try deleting *Information Technology* → friendly referential-integrity message (it still has courses).
6. **Reports** → course enrolment summary → **Export CSV** and open the file in Excel.
7. Show the unit-test run in Visual Studio Test Explorer (all green).

## Repository Expectations
Use meaningful commits (`Add student e-mail uniqueness check`, not `update`), feature branches (`feature/students-form`, `feature/reports`, `test/validators`), pull requests into `main` and a tagged release `v1.0`. See `10_Git_and_Version_Control/Git_Evidence_Guide.md`.
