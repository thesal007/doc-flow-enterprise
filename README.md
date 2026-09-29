# DocFlow Enterprise

> An enterprise document management and approval workflow platform built with **NestJS, TypeScript, Prisma, PostgreSQL, Next.js, and Domain-Driven Design (DDD)**.

## 📌 Overview

**DocFlow Enterprise** is a full-stack enterprise platform designed to manage organizational documents and automate document approval processes.

The project simulates a real-world business application where organizations need to:

* Manage users and teams
* Organize documents and folders
* Upload and manage document files
* Control access using roles and permissions
* Create document approval workflows
* Track document versions
* Approve or reject documents
* Send real-time notifications
* Maintain audit logs
* Process background tasks
* Communicate between services through events

The main goal of this project is not simply to build another CRUD application.

The goal is to practice how a **production-oriented enterprise application** can be designed, developed, tested, deployed, and maintained.

---

# 🎯 Project Purpose

Many organizations manage important documents such as:

* Contracts
* Invoices
* Policies
* Agreements
* Reports
* Customer documents
* Internal company documents

A simple file storage system is not enough for these use cases.

For example:

```text
Employee
   ↓
Uploads Contract
   ↓
Manager Reviews
   ↓
Manager Approves
   ↓
Finance Reviews
   ↓
Finance Approves
   ↓
Document Completed
```

The organization needs to know:

* Who uploaded the document?
* Who can access it?
* Who approved it?
* When was it approved?
* What changed?
* Which version is the latest?
* Who rejected it?
* Why was it rejected?
* Which workflow step is currently active?

DocFlow Enterprise is designed to solve these problems.

---

# 🏢 Business Scenario

Imagine a company with several departments:

```text
Company
│
├── Human Resources
├── Finance
├── Legal
├── Sales
└── Management
```

The company stores thousands of documents.

For example:

```text
Legal/
├── Contracts/
│   ├── Customer-A.pdf
│   ├── Customer-B.pdf
│   └── Customer-C.pdf
│
└── Agreements/

Finance/
├── Invoices/
└── Reports/

HR/
├── Employee/
└── Policies/
```

A document may require multiple approvals.

For example:

```text
Employee
   │
   │ Submit
   ▼
Manager
   │
   ├── Reject ──────→ Draft
   │
   └── Approve
          │
          ▼
       Finance
          │
          ├── Reject → Draft
          │
          └── Approve
                 │
                 ▼
              Completed
```

DocFlow manages this entire lifecycle.

---

# 🧩 Core Features

## 1. Authentication

Users can:

* Register
* Login
* Logout
* Refresh access tokens
* Verify email
* Reset passwords
* Change passwords

Authentication is responsible for establishing the identity of the user.

```text
User
 ↓
Login
 ↓
Authentication
 ↓
Access Token
 ↓
Protected API
```

---

# 2. Organization Management

Users belong to organizations.

```text
Organization
│
├── Members
├── Teams
├── Roles
└── Documents
```

Organizations can manage:

* Members
* Teams
* Roles
* Permissions
* Invitations

---

# 3. Role-Based Access Control

The system supports permission-based authorization.

Example roles:

```text
SUPER_ADMIN
ORG_ADMIN
MANAGER
MEMBER
VIEWER
```

Permissions can be more specific:

```text
document:create
document:read
document:update
document:delete
document:approve
document:share
```

Instead of simply asking:

```text
Is this user an admin?
```

the application can ask:

```text
Does this user have document:approve permission?
```

This makes authorization more flexible.

---

# 4. Document Management

Users can manage documents and folders.

Supported operations include:

* Create folder
* Upload document
* Rename document
* Move document
* Copy document
* Delete document
* Restore document
* Download document
* Share document
* Add metadata
* Add tags

Example:

```text
Company Documents
│
├── Contracts
│   ├── contract-a.pdf
│   └── contract-b.pdf
│
├── Finance
│   └── invoice-2026.pdf
│
└── HR
    └── employee-policy.pdf
```

---

# 5. Document Versioning

Documents can have multiple versions.

Example:

```text
Contract.pdf

v1
 ↓
v2
 ↓
v3
```

Each version can store:

```text
Version
├── version number
├── file
├── created by
├── created at
└── change description
```

This prevents important document history from being lost.

---

# 6. Approval Workflow

One of the main features of DocFlow is the workflow engine.

A workflow describes how a document moves through different approval stages.

Example:

```text
Draft
  ↓
Submitted
  ↓
Manager Approval
  ↓
Finance Approval
  ↓
Completed
```

A workflow can also contain branches:

```text
             Submit
                │
                ▼
             Manager
             /      \
            /        \
       Approve      Reject
          │            │
          ▼            ▼
       Finance        Draft
```

This introduces real domain rules into the application.

---

# 7. Approval Tasks

When a document reaches an approval step, the system creates an approval task.

Example:

```text
Approval Task

Document: Contract.pdf
Approver: John
Step: Finance Approval
Status: Pending
```

The approver can:

```text
APPROVE
REJECT
```

If rejected, the user can provide a reason.

---

# 8. Domain Events

Important business actions generate domain events.

Examples:

```text
DocumentCreated
DocumentSubmitted
DocumentApproved
DocumentRejected
DocumentDeleted
WorkflowCompleted
```

For example:

```text
DocumentApproved
       │
       ├── Notification
       ├── Audit Log
       └── Workflow
```

This allows different parts of the system to react to business events without tightly coupling everything together.

---

# 9. Transactional Outbox

DocFlow uses the **Transactional Outbox Pattern** for reliable event publishing.

Instead of:

```text
Save Document
      ↓
Publish Event
```

the application performs both database operations within the same transaction:

```text
Database Transaction
│
├── Save Document
│
└── Save Outbox Event
```

Then an outbox relay publishes the event:

```text
PostgreSQL
    │
    ▼
Outbox Event
    │
    ▼
Outbox Relay
    │
    ▼
NATS
    │
    ├── Notification
    ├── Audit
    └── Workflow
```

This reduces the risk of losing events when a database operation succeeds but message publishing fails.

---

# 10. Notifications

Users receive notifications when important events occur.

Examples:

```text
Your document has been approved.

John assigned a document approval task to you.

Your document was rejected.

Document "Contract.pdf" has completed approval.
```

Notifications can be delivered through:

* In-app notifications
* WebSocket
* Email

---

# 11. Audit Logs

Important actions are recorded.

Example:

```text
Audit Log

User:
John

Action:
DOCUMENT_APPROVED

Document:
Contract.pdf

Previous Status:
PENDING

New Status:
APPROVED

Timestamp:
2026-09-25 10:30:00
```

Audit logs help organizations understand what happened inside the system.

---

# 12. Background Processing

Some operations should not block the HTTP request.

For example:

```text
Upload PDF
     ↓
API
     ↓
Queue
     ↓
Worker
     ├── Extract metadata
     ├── Generate thumbnail
     ├── Extract text
     └── Index document
```

DocFlow uses background workers for long-running operations.

---

# 13. Search

Users should be able to search documents.

Example:

```text
Search:
"customer contract"
```

Possible filters:

```text
Filename
Owner
Department
Status
Created date
Tags
Document type
```

The search system can initially use PostgreSQL and later evolve toward a dedicated search engine.

---

# 🏗️ Architecture

The initial version uses a **Modular Monolith**.

```text
                    ┌──────────────┐
                    │   Next.js    │
                    │   Frontend   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   NestJS     │
                    │     API      │
                    └──────┬───────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
   PostgreSQL            Redis              NATS
        │
        ▼
      Prisma
```

The backend is divided into bounded contexts:

```text
NestJS API
│
├── Identity
├── Organization
├── Document
├── Workflow
├── Notification
└── Audit
```

---

# 🧠 Domain-Driven Design

DocFlow is designed using **Domain-Driven Design principles**.

Each major business area has its own domain.

```text
Identity
Organization
Document
Workflow
Notification
Audit
```

Inside a domain:

```text
document/
│
├── domain/
│   ├── entities/
│   ├── value-objects/
│   ├── repositories/
│   ├── services/
│   └── events/
│
├── application/
│   ├── commands/
│   ├── queries/
│   ├── handlers/
│   └── dto/
│
├── infrastructure/
│   ├── persistence/
│   ├── repositories/
│   └── messaging/
│
└── presentation/
    └── controllers/
```

The dependency direction is:

```text
Presentation
      ↓
Application
      ↓
Domain
      ↑
Infrastructure
```

The domain should not depend directly on:

```text
Prisma
PostgreSQL
NATS
Redis
HTTP
NestJS
```

Instead, infrastructure implements interfaces defined by the domain.

---

# 🛠️ Technology Stack

## Frontend

* Next.js
* React
* TypeScript
* Tailwind CSS
* shadcn/ui

## Backend

* NestJS
* TypeScript
* REST API
* WebSocket
* Swagger/OpenAPI

## Architecture

* Domain-Driven Design
* Clean Architecture principles
* Modular Monolith
* CQRS concepts
* Domain Events
* Transactional Outbox

## Database

* PostgreSQL
* Prisma ORM

## Infrastructure

* Redis
* NATS
* BullMQ
* S3 / MinIO
* Docker

## Testing

* Jest
* Supertest
* Integration Testing
* E2E Testing

## DevOps

* Docker
* CI/CD
* GitHub Actions or GitLab CI

---

# 📁 Monorepo Structure

The project uses Nx to manage the monorepo.

```text
docflow-enterprise/
│
├── apps/
│   ├── api/
│   │   └── NestJS
│   │
│   ├── web/
│   │   └── Next.js
│   │
│   └── worker/
│       └── BullMQ
│
├── libs/
│   ├── contracts/
│   ├── database/
│   ├── config/
│   └── shared/
│
├── prisma/
│   ├── schema.prisma
│   └── migrations/
│
├── infrastructure/
│   └── docker/
│
├── docs/
│   ├── architecture/
│   ├── api/
│   └── diagrams/
│
├── docker-compose.yml
├── nx.json
├── package.json
└── tsconfig.base.json
```

---

# 🔄 Example Document Lifecycle

A typical document lifecycle looks like this:

```text
                    ┌──────────────┐
                    │    Draft     │
                    └──────┬───────┘
                           │
                        Submit
                           │
                           ▼
                    ┌──────────────┐
                    │   Pending    │
                    │   Approval   │
                    └──────┬───────┘
                           │
                    ┌──────┴──────┐
                    │             │
                 Approve        Reject
                    │             │
                    ▼             ▼
              Next Approval      Draft
                    │
                    ▼
               Completed
```

Each state transition is controlled by domain rules.

For example:

```text
Draft → Submitted
Submitted → Approved
Submitted → Rejected
Approved → Completed
```

The domain should prevent invalid transitions.

---

# 🔐 Security

The project will practice common enterprise security patterns:

* JWT authentication
* Refresh tokens
* Password hashing
* Role-based access control
* Permission-based authorization
* Resource ownership checks
* Input validation
* Rate limiting
* Secure file access
* Audit logging
* API authentication
* Environment-based secrets

---

# 🧪 Testing Strategy

Testing will happen at multiple levels.

### Unit Tests

Test domain behavior:

```text
Document.submit()
Document.approve()
Document.reject()
Workflow.canApprove()
```

### Integration Tests

Test:

```text
Application
     ↓
Repository
     ↓
PostgreSQL
```

### E2E Tests

Test:

```text
HTTP Request
     ↓
Controller
     ↓
Application
     ↓
Domain
     ↓
Database
```

---

# 🐳 Local Development

The infrastructure will eventually run through Docker:

```text
Docker Compose
│
├── PostgreSQL
├── Redis
├── NATS
└── MinIO
```

The application itself can run with Nx:

```bash
nx serve api
nx serve web
nx serve worker
```

---

# 📈 Development Roadmap

## Phase 1 — Foundation

* [ ] Nx monorepo
* [ ] NestJS API
* [ ] Next.js frontend
* [ ] PostgreSQL
* [ ] Prisma
* [ ] Docker
* [ ] Environment configuration

## Phase 2 — Identity

* [ ] Registration
* [ ] Login
* [ ] JWT
* [ ] Refresh tokens
* [ ] User profile
* [ ] Password management

## Phase 3 — Organization

* [ ] Organizations
* [ ] Members
* [ ] Teams
* [ ] Roles
* [ ] Permissions
* [ ] Invitations

## Phase 4 — Documents

* [ ] Folders
* [ ] Documents
* [ ] Upload
* [ ] Download
* [ ] File metadata
* [ ] Soft delete
* [ ] Restore
* [ ] Versioning

## Phase 5 — Workflow

* [ ] Workflow definitions
* [ ] Workflow steps
* [ ] Workflow instances
* [ ] Approval tasks
* [ ] Approve
* [ ] Reject
* [ ] Workflow state transitions

## Phase 6 — Events

* [ ] Domain events
* [ ] Event handlers
* [ ] NATS
* [ ] Transactional Outbox
* [ ] Outbox relay

## Phase 7 — Platform Features

* [ ] Redis
* [ ] BullMQ
* [ ] WebSocket
* [ ] Notifications
* [ ] Audit logs
* [ ] Search
* [ ] S3/MinIO

## Phase 8 — Quality

* [ ] Unit tests
* [ ] Integration tests
* [ ] E2E tests
* [ ] API documentation
* [ ] Error handling
* [ ] Logging
* [ ] Monitoring
* [ ] CI/CD

---

# 🎓 Learning Objectives

The main purpose of this project is to develop strong practical skills in:

### Backend Development

* NestJS
* TypeScript
* REST API
* Authentication
* Authorization
* Database design
* Transactions
* File handling
* Background processing

### Software Architecture

* DDD
* Clean Architecture
* SOLID
* Modular Monolith
* CQRS
* Domain Events
* Event-driven architecture
* Transactional Outbox
* Eventually distributed services

### Database

* PostgreSQL
* Prisma
* Relations
* Transactions
* Indexes
* Constraints
* Migrations
* Optimistic locking

### Distributed Systems

* NATS
* Message delivery
* Event processing
* Idempotency
* Outbox pattern
* Background workers
* Caching

### Professional Development

* Testing
* Docker
* CI/CD
* API documentation
* Git
* Code reviews
* Architecture documentation
* Observability

---

# 🚀 Long-Term Goal

The project starts as a **modular monolith**.

```text
Next.js
   ↓
NestJS
   ↓
PostgreSQL
```

As the system becomes more complex, selected components can be extracted into independent services.

```text
                         API
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
      Identity        Document         Workflow
       Service         Service          Service
          │               │               │
          └───────────────┼───────────────┘
                          │
                         NATS
                          │
                ┌─────────┴─────────┐
                ▼                   ▼
          Notification            Search
             Service              Service
```

This approach allows the architecture to evolve based on actual system requirements instead of introducing distributed-system complexity before it is needed.

---

# 💡 What This Project Represents

DocFlow Enterprise is intentionally more than a portfolio CRUD application.

It is a learning project for understanding how to build a **large-scale, maintainable web application** from the domain model through infrastructure and deployment.

The project focuses on the complete engineering lifecycle:

```text
Business Requirements
        ↓
Domain Modeling
        ↓
Architecture
        ↓
Implementation
        ↓
Database
        ↓
Events
        ↓
Testing
        ↓
Infrastructure
        ↓
Deployment
        ↓
Monitoring
```

The ultimate objective is to understand not only **how to write code**, but also **how different parts of a production application work together**.

---

# 📚 Documentation

Project documentation will be maintained under:

```text
docs/
├── architecture/
├── api/
├── database/
├── domain/
├── deployment/
└── diagrams/
```

Architecture diagrams should document:

* System architecture
* Domain boundaries
* Database relationships
* Document lifecycle
* Workflow lifecycle
* Event flow
* Outbox flow
* Authentication flow
* Deployment architecture

---

# 👨‍💻 Project Status

> 🚧 **In Development**

DocFlow Enterprise is being developed incrementally, starting with a modular monolith and gradually introducing event-driven and distributed-system concepts.

---

# 📄 License

This project is currently intended as a learning and portfolio project.
