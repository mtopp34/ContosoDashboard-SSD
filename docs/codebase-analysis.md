# ContosoDashboard - Codebase Analysis

## ContosoDashboard Application

ContosoDashboard is an **ASP.NET Core 8.0 Blazor Server** training application that simulates an internal employee productivity dashboard for the fictional Contoso corporation. It is designed for **offline/local use** with SQL Server LocalDB and has a migration path to Azure.

---

### Application Features

| Feature | Description |
|---------|-------------|
| **Dashboard Home** | Personalized summary cards (active tasks, due today, projects, unread notifications), announcements feed, quick actions |
| **Task Management** | View/filter tasks by status, priority, and project; inline status updates; priority color-coding; overdue highlighting |
| **Project Management** | Browse projects with completion percentages, team member visibility, status badges |
| **Project Details** | Individual project view with task list, team members, and statistics |
| **Team Directory** | Browse team members by department with availability status and contact info |
| **Notifications** | Read/unread notifications with priority badges and multiple notification types |
| **User Profile** | Edit personal info, manage availability status, configure notification preferences |
| **Auth & Authorization** | Cookie-based mock login, role-based policies (Employee, TeamLead, ProjectManager, Administrator), IDOR protection at the service layer |

---

### Tech Stack

| Layer | Technology |
|-------|-----------|
| **Framework** | ASP.NET Core 8.0 with Blazor Server |
| **Database** | SQL Server LocalDB + Entity Framework Core 8.0 |
| **UI** | Bootstrap 5.3 + Bootstrap Icons |
| **Authentication** | Cookie-based mock authentication (training purposes) |
| **Authorization** | Claims-based identity with role-based policies |
| **Language** | C# 12 with nullable reference types |

---

### Architecture

The application follows a clean N-tier architecture with interface-based dependency injection and defense-in-depth security.

#### Key Directories

| Directory | Purpose |
|-----------|---------|
| `ContosoDashboard/Models/` | Entity classes (User, TaskItem, Project, Notification, TaskComment, ProjectMember, Announcement) |
| `ContosoDashboard/Services/` | Business logic with authorization checks (UserService, TaskService, ProjectService, NotificationService, DashboardService) |
| `ContosoDashboard/Pages/` | Blazor/Razor UI pages (Dashboard, Tasks, Projects, Team, Notifications, Profile, Login/Logout) |
| `ContosoDashboard/Data/` | EF Core DbContext with seed data, relationships, and indexes |
| `ContosoDashboard/Shared/` | Layout components (MainLayout, NavMenu, RedirectToLogin) |

#### Data Models

| Model | Key Fields |
|-------|-----------|
| **User** | UserId, Email, DisplayName, Department, JobTitle, Role (Employee/TeamLead/ProjectManager/Administrator), AvailabilityStatus |
| **TaskItem** | TaskId, Title, Description, Priority (Low/Medium/High/Critical), Status (NotStarted/InProgress/Completed), DueDate, AssignedUserId, ProjectId |
| **Project** | ProjectId, Name, Description, ProjectManagerId, Status (Planning/Active/OnHold/Completed), CompletionPercentage |
| **Notification** | NotificationId, UserId, Title, Message, Type, Priority, IsRead |
| **TaskComment** | CommentId, TaskId, UserId, CommentText |
| **ProjectMember** | ProjectMemberId, ProjectId, UserId, Role |
| **Announcement** | AnnouncementId, Title, Content, PublishDate, ExpiryDate, IsActive |

#### Security Implementation

- **Authentication**: Cookie-based with 8-hour sliding expiration
- **Authorization**: Attribute-based (`[Authorize]`) + service-level checks
- **IDOR Prevention**: Service methods validate user access before returning data
- **Security Headers**: CSP, X-Frame-Options, X-XSS-Protection, X-Content-Type-Options, Referrer-Policy
- **Defense in Depth**: Middleware security headers + service-layer validation

#### Seed Data

The database auto-creates on startup with:
- 4 users (Administrator, Project Manager, Team Lead, Employee)
- 1 project with 2 project members
- 3 sample tasks
- 1 announcement

---

## GitHub Spec Kit (`.specify/` and `.github/`)

**SpecKit** is a **Spec-Driven Development (SDD) framework** for structured, AI-assisted feature development. It enforces a specification-first workflow where every feature goes through a defined pipeline before code is written.

---

### Workflow Pipeline

```
User Description
  -> /speckit.specify    -> creates spec.md
  -> /speckit.clarify    -> refines spec.md with targeted questions
  -> /speckit.plan       -> creates plan.md + supporting artifacts
  -> /speckit.tasks      -> creates tasks.md (dependency-ordered)
  -> /speckit.analyze    -> validates consistency across all artifacts (read-only)
  -> /speckit.implement  -> executes tasks from tasks.md
  -> /speckit.checklist  -> generates quality checklists
```

---

### `.specify/` Directory Contents

| Path | Purpose |
|------|---------|
| `memory/constitution.md` | Project principles and non-negotiable rules that all specs/plans must comply with |
| `templates/spec-template.md` | Blueprint for feature specifications (user stories, requirements, success criteria) |
| `templates/plan-template.md` | Blueprint for implementation plans (tech context, constitution checks, project structure) |
| `templates/tasks-template.md` | Blueprint for task lists organized by user story with parallelism markers |
| `templates/checklist-template.md` | Blueprint for quality checklists |
| `templates/agent-file-template.md` | Blueprint for AI agent context files (CLAUDE.md, GEMINI.md, etc.) |
| `scripts/powershell/check-prerequisites.ps1` | Validates prerequisites for each workflow phase |
| `scripts/powershell/common.ps1` | Shared utility functions (repo root, branch detection, path helpers) |
| `scripts/powershell/create-new-feature.ps1` | Creates new feature branches and initializes spec.md |
| `scripts/powershell/setup-plan.ps1` | Initializes plan.md from template |
| `scripts/powershell/update-agent-context.ps1` | Updates AI agent context files for 12+ coding assistants |

---

### `.github/` Directory (Agents & Prompts)

The `.github/agents/` and `.github/prompts/` directories contain **9 agent/prompt pairs** that define how AI assistants handle each workflow phase:

| Agent | Purpose | Input | Output |
|-------|---------|-------|--------|
| `speckit.specify` | Create feature specification | User description | `spec.md` |
| `speckit.clarify` | Refine requirements | spec.md + user answers | Updated `spec.md` |
| `speckit.plan` | Create implementation plan | `spec.md` | `plan.md`, research.md, data-model.md, contracts/ |
| `speckit.tasks` | Generate task list | spec.md + plan.md | `tasks.md` |
| `speckit.analyze` | Quality analysis (read-only) | spec.md + plan.md + tasks.md | Inconsistency report |
| `speckit.checklist` | Generate checklists | Feature context | `checklist.md` |
| `speckit.implement` | Execute implementation | All artifacts | Code changes |
| `speckit.constitution` | Constitutional enforcement | Constitution rules | Compliance checks |
| `speckit.taskstoissues` | Convert tasks to GitHub issues | `tasks.md` | GitHub issues with labels and milestones |

---

### Key Concepts

#### Constitution

A governance document (`memory/constitution.md`) defining non-negotiable project rules. The `/speckit.analyze` agent validates all artifacts against it. It includes:

- **Core Principles**: Project's non-negotiable rules
- **Additional Constraints**: Technology, security, and performance standards
- **Development Workflow**: Code review, testing gates, deployment policies
- **Governance**: Amendment process and compliance verification

#### Templates

Templates in `.specify/templates/` produce feature artifacts stored under `specs/###-feature-name/`:

- `spec-template.md` -> `spec.md` (priority-ordered user stories with test criteria)
- `plan-template.md` -> `plan.md` (technical approach, dependencies, constitution checks)
- `tasks-template.md` -> `tasks.md` (phased tasks: Setup -> Foundational -> User Stories -> Polish)
- `checklist-template.md` -> `checklist.md` (quality validation items)

#### Task Organization

Tasks in `tasks.md` follow a structured format:

- `[ID]` - Task identifier (T001, T002, etc.)
- `[P?]` - Parallelism marker (`[P]` = can run in parallel)
- `[Story]` - User story reference (US1, US2, US3)
- Phases: Setup -> Foundational -> User Stories (P1/P2/P3) -> Polish
- Tests-first pattern: tests are written before implementation

#### PowerShell Automation

Scripts automate the workflow:

- **create-new-feature.ps1**: Creates branch `###-feature-name`, initializes `spec.md`
- **setup-plan.ps1**: Copies plan template to feature directory
- **check-prerequisites.ps1**: Validates required artifacts exist before each phase
- **update-agent-context.ps1**: Updates context files for Claude, Copilot, Cursor, Windsurf, Gemini, Qwen, and more

#### Multi-Agent Support

The `update-agent-context.ps1` script maintains context files for 12+ AI coding assistants:

- `CLAUDE.md` (Claude Code)
- `GEMINI.md` (Gemini)
- `.github/agents/copilot-instructions.md` (GitHub Copilot)
- `.cursor/rules/specify-rules.mdc` (Cursor)
- `.windsurf/rules/specify-rules.md` (Windsurf)
- And others (Kilocode, Auggie, Roo, CodeBuddy, Qoder, etc.)
