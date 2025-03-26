# Cake Order Management System

### Technical Specification – Initial Draft

---

## Introduction

This document outlines the **initial technical specification** for the Cake Order Management System, which will be implemented primarily as a Telegram bot-based solution. It includes architecture, technology stack, implementation details, and deployment strategy.

> Note:
> 
> 
> This is a **living document** and is subject to change throughout the project lifecycle. As we gain more clarity from business stakeholders or encounter technical trade-offs, this document will be updated accordingly.
> 

---

## Project Management & Collaboration

### Communication with PM and Instructors

- Developers are **required to communicate proactively** with the **Project Manager (PM)** and **Technical Lead (Instructor)** regarding any unclear or missing technical or business requirements.
- When architectural trade-offs or difficult implementation decisions arise, they will be discussed with instructors to **deepen the team's understanding** and ensure we make sound decisions.

### Team Dynamics

- It is **expected that other students may join** the project over time. As such, writing clear, modular, and well-documented code is essential.
- Contributors should be able to onboard quickly with minimal friction.

### Instructor & PM Role

- Instructors will step in to help **validate architectural decisions**, guide on engineering best practices, and help explore alternative solutions when needed.
- The PM’s responsibility is to:
    - Document business and technical requirements.
    - Coordinate communication between engineering and stakeholders.
    - Ensure all questions are answered clearly and promptly.
    - Provide **mockups, wireframes, or visual references** to guide implementation and reduce ambiguity.

---

## Technology Stack

### Backend

| Category | Technology | Rationale |
| --- | --- | --- |
| Language | Python 3.10+ | Modern features, strong typing, widespread ecosystem |
| Package Manager | Poetry | Deterministic builds, better management than pip/requirements.txt |
| Web Framework | FastAPI | Async support, built-in documentation, fast performance |
| ORM | SQLAlchemy | Mature, flexible, handles complex relations |
| Migrations | Alembic | Integrates well with SQLAlchemy for DB schema versioning |
| Telegram Framework | aiogram | Async-native, well-maintained, good state management |
| Documentation | OpenAPI/Swagger | Auto-generated via FastAPI for easier testing and understanding |

### Database

| Component | Technology | Rationale |
| --- | --- | --- |
| Primary DB | PostgreSQL 14+ | Strong relational support, JSON fields, geolocation support |
| Media Storage | DO Spaces | S3-compatible, scalable, cost-effective |

### Infrastructure

| Area | Technology | Rationale |
| --- | --- | --- |
| Containerization | Docker | Portable environments, consistency across systems |
| CI/CD | GitHub Actions | Direct GitHub integration, flexible and easy setup |
| Hosting | DigitalOcean | Affordable, flexible setup via Droplets or App Platform |
| Payments | Stripe, Telegram | Multiple options, flexible, secure APIs |

---

## System Architecture

```
pgsql
CopyEdit
┌─────────────────┐     ┌───────────────────┐     ┌────────────────┐
│  Telegram Bot   │────>│  Application API  │────>│ Database Layer │
└─────────────────┘     └───────────────────┘     └────────────────┘
        │                        │                        │
        ▼                        ▼                        ▼
┌─────────────────┐     ┌───────────────────┐     ┌────────────────┐
│ Notification    │     │  Business Logic   │     │ Asset Storage  │
│ Service         │     │                   │     │ (DO Spaces)    │
└─────────────────┘     └───────────────────┘     └────────────────┘
                                 │
                                 ▼
                        ┌───────────────────┐
                        │  Payment Service  │
                        │  (Stripe/Telegram)│
                        └───────────────────┘

```

---

## Component Communication

### Telegram Bot → Application API

- Uses RESTful HTTP requests
- Authenticated via API tokens
- Manages state locally, persists data via backend API

**Why:** Keeps Telegram logic lightweight and allows reuse of business logic via other interfaces later (e.g., web, mobile).

### Application API → Business Logic

- Validates and passes data to service layer
- Service returns processed output

**Why:** Clean separation improves testability and modularity.

### Business Logic → Database

- Repository pattern to abstract DB queries
- Services manage transactions and workflows

**Why:** Ensures logic isn’t tied to DB internals; supports mocking for tests.

---

## Project Structure

```
pgsql
CopyEdit
cake_order_system/
├── pyproject.toml
├── poetry.lock
├── alembic/
├── app/
│   ├── api/
│   ├── bot/
│   ├── core/
│   ├── db/
│   ├── storage/
│   ├── payments/
│   └── notifications/
├── scripts/
├── tests/
├── docker/
└── main.py

```

---

## Data Models

### User

- id, telegram_id, name, email, phone, verified, role

### Baker (inherits User)

- bakery_id, bio, is_approved, approval_date

### Customer (inherits User)

- dietary_preferences, allergies, default_address

### Bakery

- name, description, location, rating, opening_hours

### Cake

- bakery_id, name, description, price, availability
- Relations: ingredients (many-to-many), images (one-to-many)

### Order

- customer_id, bakery_id, cake_id, status, delivery date, total price
- One-to-one relation with payment

### Asset

- file_path, file_type, linked object type and ID, upload timestamp

---

## Feature Highlights

### Authentication

- Use Telegram ID for login
- Fields reserved for email/SMS in future versions

### Media Management

- Upload via Telegram → stored in DO Spaces
- File metadata stored in DB

### Notification System

- Telegram for MVP
- Scalable to email/SMS via pluggable architecture

### Payments

- Abstracted layer to support both Telegram and Stripe
- Choice depends on use case, fees, and UX

### Baker Approval Workflow

- Admin panel for approval
- Tracks approval status, metadata

---

## Deployment Strategy

### Option 1: DigitalOcean Droplet (MVP)

- Dockerized services on single droplet
- NGINX reverse proxy
- Let’s Encrypt SSL
- PostgreSQL container

**Why:** Affordable, simple, easy to control in MVP phase.

### Option 2: DO App Platform (Future)

- Managed Postgres
- Autoscaling and containerized services

**Why:** Suitable for future scale-up when traffic justifies it.

---

### CI/CD Pipeline

- Build/test with GitHub Actions
- Push Docker images
- Auto-deploy on main branch push

---

## Implementation Timeline

| Phase | Timeline | Deliverables |
| --- | --- | --- |
| Infrastructure Setup | Week 1 | Project layout, DB setup, CI/CD |
| Core Services | Weeks 2–3 | Models, Repos, Business Logic, API |
| Telegram Bot | Weeks 4–5 | User flows, ordering system |
| Notifications & Payments | Week 6 | Notification service, payment gateway |
| Testing & Deployment | Week 7 | QA, UAT, Production deployment |

---

## Maintenance & Monitoring

- **Error Tracking**: Sentry or equivalent
- **Performance Monitoring**: DigitalOcean Monitoring
- **Backups**: Scheduled DB backups
- **Ops Documentation**: Runbooks for deployment, recovery

# Next Steps & Responsibilities

## 1. Project Kickoff & Roles

### 🔹 Instructor

- Set up **project repository** (private GitHub repo).
- Configure **initial folder structure**, push base FastAPI/Poetry project.
- Invite **student/intern** as a collaborator.
- Prepare and **provision server environment** (DigitalOcean Droplet or App Platform).
- Share required **logins** (DigitalOcean, Sentry, Stripe sandbox, etc.) in a secure channel (e.g., 1Password, Notion vault).

### 🔹 Project Manager (PM)

- Finalize and upload the **technical specification** document.
- Upload **mockups / wireframes** (Figma or image format).
- Create a **Notion workspace or Kanban board** for project management.
- Schedule regular check-in meetings.
- Track **services cost estimates** and report them:
    - DigitalOcean (Droplet, Spaces)
    - Stripe/Telegram Payments (fees per transaction)
    - Email/SMS services (if used later)
    - Sentry (error tracking – free tier should suffice for MVP)

### 🔹 Student / Intern

- Attend **onboarding meeting** with Instructor + PM.
- Read and understand the **Technical Specification** and architecture.
- Familiarize with:
    - Technology stack (Poetry, FastAPI, aiogram, SQLAlchemy, Docker)
    - Folder structure and project layout
    - Code quality and branching expectations (GitHub Flow)
- Perform the following **action items**:
    - [ ]  Clone the repository
    - [ ]  Set up local environment (install dependencies via Poetry)
    - [ ]  Run the basic app locally (FastAPI + sample endpoint)
    - [ ]  Review existing database models and repository interfaces
    - [ ]  Get access to DO Spaces or similar (if working with asset upload)
    - [ ]  Understand and test Telegram bot basics

---

## 2. Near-Term Decisions to Be Made

| Topic | Who Decides | Notes |
| --- | --- | --- |
| Payment method for MVP | Instructor + PM | Stripe vs Telegram (based on dev effort & fees) |
| Notification service priority | Instructor + PM | Start with Telegram; email/sms later |
| Admin panel tool | Instructor | Could be basic FastAPI endpoints or lightweight UI |
| Cost tracking sheet location | PM | Google Sheets or Notion preferred |
| Project domain (if any) | PM | Optional for MVP, else subdomain from instructor |

---

## 3. Meeting & Check-In Plan

| Meeting Type | Attendees | Purpose | Frequency |
| --- | --- | --- | --- |
| Onboarding Session | Student, Instructor, PM | Project walkthrough, Q&A | Once (Week 1) |
| Weekly Tech Sync | Student, Instructor | Review progress, unblock development | Weekly |
| Stakeholder Check-in | Instructor, PM | Validate features, align on scope | Bi-weekly |
| Ad Hoc Debug/Support | Student, Instructor | As needed for technical issues | As needed |

---

## 4. Cost Awareness & Tools Used

| Service | Purpose | Est. Monthly Cost (USD) | Notes |
| --- | --- | --- | --- |
| DigitalOcean Droplet | App + DB hosting | $6–12 | Start with 2GB RAM option |
| DO Spaces | Media asset storage | $5+ (based on usage) | CDN included, scalable |
| Sentry | Error tracking | Free tier | Upgrade if needed later |
| Stripe / Telegram Pay | Payments | Based on transactions | Keep logs of fees per transaction |
| GitHub | Repo + CI/CD | Free tier (private repo) | GitHub Actions limited to 2K mins |
| Email/SMS (Future) | Notifications | TBD | Optional in MVP |

---