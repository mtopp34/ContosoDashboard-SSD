# Implementation Plan: Document Upload and Management

**Branch**: `001-document-management` | **Date**: 2026-02-03 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-document-management/spec.md`

## Summary

Implement a centralized document management system within Contoso Dashboard enabling employees to upload, organize, search, and share work-related documents with role-based access control. The system uses local SQLite storage with a service abstraction layer (IFileStorageService) enabling future Azure Blob Storage migration. Core MVP includes upload/download, metadata management, search with substring matching, and persistent notifications. Training-appropriate with offline capability, stub virus scanning, and security-first RBAC.

## Technical Context

**Language/Version**: C# 12 (.NET 8.0)
**Primary Dependencies**: ASP.NET Core 8.0, Blazor Server, Entity Framework Core 8.0, Microsoft.EntityFrameworkCore.Sqlite
**Storage**: SQLite (local filesystem via EF Core)
**Testing**: xUnit or MSTest (determined in Phase 1)
**Target Platform**: Windows, macOS, Linux (cross-platform via .NET 8.0)
**Project Type**: Web application (Blazor Server backend + frontend pages)
**Performance Goals**: Upload <30s (25MB), Search <2s, Preview <3s, List <2s (500 docs)
**Constraints**: Offline-capable, no external services, local-only, GUID-based file naming, interface-abstracted storage
**Scale/Scope**: Training context (10-100 concurrent users assumed), up to 500 documents per user, 6 categories, 4 roles

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Details |
|-----------|--------|---------|
| **I. Training-First Design** | ✅ PASS | Feature is self-contained, runnable offline, no external accounts required. Mock virus scanning implemented as stub. |
| **II. Offline-First Architecture** | ✅ PASS | Uses SQLite locally, local filesystem for documents, cookie-based mock auth, IFileStorageService abstraction for future cloud migration. |
| **III. Security by Design** | ✅ PASS | [Authorize] attributes on pages, IDOR prevention via service layer auth checks, RBAC with 4-level hierarchy (Employee < TeamLead < PM < Admin), security headers via middleware. |
| **IV. Spec-Driven Development** | ✅ PASS | Following SpecKit workflow: specify → clarify → plan → tasks → implement. 7 user stories prioritized P1-P3, independently testable. |
| **V. Simplicity and YAGNI** | ✅ PASS | No speculative features, stub virus scanning instead of real integration, basic search (substring not full-text), minimal dependencies. |

**Gate Result**: ✅ PASS - All principles satisfied. No violations. Ready for Phase 0.

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
src/ContosoDashboard/
├── Data/
│   ├── Models/
│   │   ├── Document.cs              # Document entity with metadata
│   │   ├── DocumentTag.cs           # Tag association entity
│   │   ├── DocumentShare.cs         # Share/permission entity
│   │   ├── DocumentActivity.cs      # Audit trail entity
│   │   └── Notification.cs          # Persistent notification entity
│   ├── ApplicationDbContext.cs       # EF Core context
│   └── Migrations/                  # EF Core migrations for documents schema
│
├── Services/
│   ├── IFileStorageService.cs       # Abstract storage interface
│   ├── LocalFileStorageService.cs   # Local filesystem implementation
│   ├── IVirusScanService.cs         # Abstract virus scanning interface
│   ├── StubVirusScanService.cs      # Training stub implementation
│   ├── DocumentService.cs           # Business logic: CRUD, search, permissions
│   ├── NotificationService.cs       # Notification creation and retrieval
│   └── ShareService.cs              # Document sharing and access control
│
├── Pages/
│   └── Documents/
│       ├── Upload.razor             # Document upload UI (Blazor page)
│       ├── MyDocuments.razor        # My Documents list with sort/filter
│       ├── DocumentDetail.razor     # Document preview, metadata edit
│       ├── SharedWithMe.razor       # Shared documents view
│       ├── Search.razor             # Document search results
│       └── ProjectDocuments.razor   # Project-scoped documents
│
├── Controllers/
│   └── DocumentsController.cs       # API endpoints for download/preview
│
└── Shared/
    └── Components/
        └── DocumentUploadModal.razor # Reusable upload component

Tests/
├── Unit/
│   ├── DocumentServiceTests.cs      # Service logic tests
│   ├── DocumentValidationTests.cs   # File validation tests
│   └── AccessControlTests.cs        # RBAC authorization tests
│
├── Integration/
│   └── DocumentWorkflowTests.cs     # End-to-end upload/search/download
│
└── Contract/
    └── DocumentApiTests.cs          # API endpoint contract tests
```

**Structure Decision**: Blazor Server web application (ASP.NET Core 8.0 with N-tier architecture). Documents feature uses service layer for business logic with interface abstractions for storage and virus scanning. Pages layer provides Blazor components for UI. Controllers handle file download/preview endpoints requiring authorization. Tests organized by type: unit (services), integration (workflows), contract (APIs).

## Phase 0: Research & Knowledge Gaps

**Status**: Ready to execute
**Prerequisite**: Constitution Check ✅ PASS

### Research Tasks

1. **Blazor File Upload Best Practices**: MemoryStream vs temp files, InputFile component state management, progress tracking patterns
2. **EF Core Relationships**: Many-to-many (DocumentTag), one-to-many (Document → User/Project), cascade delete behavior
3. **PDF/Image Preview in Blazor**: Browser-native vs library (e.g., PDFjs), security implications (XSS), CORS for embedded content
4. **Search Implementation**: EF Core LINQ substring matching (EF.Functions.Like), performance with large result sets, authorization in queries
5. **Local File Storage Patterns**: Path traversal prevention (GUID-based names), permission-based access to disk files, cleanup on delete

### Research Output

File: `research.md` (to be generated during Phase 0)
- Decision: [selected pattern]
- Rationale: [why chosen]
- Alternatives considered: [evaluated options]

---

## Phase 1: Design & Contracts

**Prerequisites**: research.md complete

### Phase 1a: Data Model

**Output**: `data-model.md`

**Entities to define**:
- **Document**: DocumentId (int), Title (string), Category (string), Description (string), FilePath (string), FileName (string), FileSize (long), FileType (string), UploadDate (DateTime), UploaderUserId (int), AssociatedProjectId (int?), User (FK), Project (FK)
- **DocumentTag**: TagId (int), DocumentId (FK), TagName (string)
- **DocumentShare**: ShareId (int), DocumentId (FK), SharedByUserId (FK), SharedWithUserId (FK), ShareDate (DateTime), IsActive (bool)
- **DocumentActivity**: ActivityId (int), DocumentId (FK), UserId (FK), Action (string), Timestamp (DateTime), Details (JSON)
- **Notification**: NotificationId (int), RecipientUserId (FK), DocumentId (FK), Message (string), NotificationType (string), IsRead (bool), CreatedDate (DateTime), ReadDate (DateTime?)

**Validation Rules**:
- Document.Title: required, max 255 chars
- Document.FileSize: max 26843545 bytes (25 MB)
- DocumentTag.TagName: required, max 50 chars, unique per Document
- DocumentShare.IsActive: soft-revoke pattern (don't delete, mark inactive)

**Relationships**:
- User → Document (1:N), User → DocumentShare (1:N)
- Project → Document (0:N), Project → DocumentActivity (0:N)
- Document → DocumentTag (1:N), Document → DocumentShare (1:N), Document → DocumentActivity (1:N)

### Phase 1b: API Contracts

**Output**: `contracts/documents-api.openapi.json`

**Key Endpoints**:
- `POST /api/documents/upload` - Upload file with metadata
- `GET /api/documents` - List my documents (with sort/filter)
- `GET /api/documents/search?q=...` - Search documents
- `GET /api/documents/{id}/download` - Download file (auth check)
- `GET /api/documents/{id}/preview` - Stream for preview (auth check, PDF/image only)
- `PATCH /api/documents/{id}` - Edit metadata
- `POST /api/documents/{id}/version` - Upload new version
- `DELETE /api/documents/{id}` - Delete document
- `POST /api/documents/{id}/share` - Share with user
- `GET /api/documents/shared-with-me` - List shared documents
- `GET /api/notifications` - Get user's notifications

### Phase 1c: Agent Context Update

**Output**: Updated `.specify/memory/agent-context.md`

- Run: `.specify/scripts/powershell/update-agent-context.ps1 -AgentType claude`
- Add technologies: Blazor Server, EF Core Sqlite, MemoryStream patterns, IFileStorageService abstraction

### Phase 1d: Quickstart

**Output**: `quickstart.md`

Instructions for:
1. Running database migrations
2. Creating sample documents via API
3. Testing upload/download/search flows
4. Verifying access controls (try as different roles)

---

## Phase 1 Validation (Pre-Phase 2)

Re-check Constitution Check after design artifacts complete:

- ✅ All data entities have security-relevant fields (UserId, Role-based filtering)
- ✅ All endpoints include [Authorize] and service-layer auth checks
- ✅ File storage uses GUID-based paths (no user input in paths)
- ✅ Storage abstraction (IFileStorageService) enables Azure migration
- ✅ Virus scanning abstraction (IVirusScanService) enables real implementation
- ✅ Search uses substring matching (no full-text indexing, stays YAGNI)
- ✅ Notifications persist in database (no real-time dependencies)

**Gate Result**: ✅ PASS - Ready for Phase 2 (task generation via /speckit.tasks)
