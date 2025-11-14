Nice. You picked C# as the main horse and Go for heavy lifting later — sensible and merciful to your future self. Below is a surgical, no-fluff MVP plan (phases + concrete deliverables) and a ready-to-run PostgreSQL schema for the core domain (tickets, attachments, CMDB, changes, KB, auth, workflows, audit). Copy-paste deployable SQL is included. No hand-waving. Let’s build the thing.

MVP scope (what you actually need to prove product/market fit)

Authentication: local + SSO (OIDC/Azure AD + LDAP/SAML later). Role-based access control.

Ticket core: create/read/update/delete, comments, assignment, statuses, priorities.

Attachments: chunked uploads → object store (MinIO), metadata in DB, dedupe by checksum.

Notifications: real-time via SignalR + email outbound + notification center.

Search: Postgres full-text (tsvector) for tickets + KB.

CMDB (basic assets) and Change Requests (basic lifecycle).

Knowledge Base with WYSIWYG content, versioning, search.

Simple workflow engine (state-machine with webhooks/actions).

Audit logs, basic SLA markers, and admin panel for RBAC + org config.

Inbound email parsing to create/update tickets (simple RFC parsing).

Observability (Prometheus metrics endpoint + Grafana dashboards).

Docker Compose single-server deployment (Postgres, MinIO, Redis, app).


High-level tech stack (MVP)

Backend: C# (.NET 8+) with ASP.NET Core, EF Core (Npgsql), SignalR, Hangfire (or BackgroundService + Redis), FluentValidation.

Frontend: React + TypeScript + Vite, react-query, Zustand, MUI/Radix UI, tiptap for editor.

DB: PostgreSQL (UUID PKs, JSONB fields, tsvector search).

Object store: MinIO (S3-compatible) on same server.

Cache/Queue: Redis (caching, pub/sub, background job queue).

Metrics: Prometheus + Grafana.

CI: GitHub Actions (build, test, produce docker images).

Deployment: Docker Compose (single server) → later Kubernetes if you scale.


Step-by-step MVP plan (phases — do these in order; each phase has clear deliverables)

I’m not giving time estimates. Do these phases in sequence; stop after any phase if you need to ship early.

Phase 0 — Setup & infra (foundation)

Deliverables: repo skeleton, base Docker Compose, infra docs.

Init mono-repo: backend/, frontend/, infra/.

Dockerfile + docker-compose.yml for app, Postgres, MinIO, Redis.

Create basic logging + structured logs (Serilog).

Add Prometheus metrics endpoint and Grafana container or dashboard template.

CI pipeline for build/test/docker image.



Phase 1 — Auth + Core models + DB

Deliverables: working login (local), user/role management, DB migrations.

EF Core models + migrations for Users, Roles, UserRoles, Organizations.

JWT-based auth + refresh tokens, secure password hashing (Argon2 or PBKDF2).

Admin UI to create users/orgs/roles.

Add SSO config storage for OIDC/SAML (no SSO flow yet).



Phase 2 — Ticket CRUD + persistence + basic UI

Deliverables: create/view/update tickets, ticket number, comments, basic filters.

Ticket model + API endpoints (paged list, single ticket, update).

Frontend: ticket list + detail + create/edit forms.

Implement server-side validation and optimistic concurrency.

Add ticket audit logging on create/update/assign/close.



Phase 3 — Attachments + object store + streaming uploads

Deliverables: chunked/resumable upload + MinIO integration + attachment metadata.

Implement chunked upload protocol (Tus or simple chunking).

Store files on MinIO, save metadata in attachments table, compute SHA-256 checksum for dedupe.

Add thumbnails/preview worker (background job).

Ensure uploads are streamed — no full-file memory buffering.



Phase 4 — Real-time + Notifications + Email

Deliverables: SignalR real-time events + notification center + basic inbound email.

SignalR hubs for ticket events; integrate with frontend by invalidating caches/react-query.

Notification table + unread counters + UI.

Simple inbound email ingestion endpoint (webhook from your mail gateway) to parse and create/update tickets.



Phase 5 — Search + KB + CMDB basics

Deliverables: tsvector search for tickets & KB; KB CRUD; assets table + UI.

Implement Postgres full-text tsvector and triggers.

KB article CRUD, versioning, and linkable to tickets.

Assets (CMDB) CRUD + basic relationships to tickets.



Phase 6 — Workflows, Change Requests, SLA basics

Deliverables: workflow definitions (JSON), workflow engine runner, change request CRUD.

Basic workflow engine: store definition JSON and step through states with triggers.

Change request model & UI (create, schedule, approve).

SLA table with simple violation detection worker.



Phase 7 — Admin features, import tool, and migration helpers

Deliverables: admin dashboards, BMC/Remedy CSV import tool, migration path.

Admin UI for system settings, connectors, export/import.

Migration assistant for CSV exports from BMC/Remedy to map fields and import batches.



Phase 8 — Hardening & observability

Deliverables: security audit checklist, backups & restores, monitoring dashboards, runbooks.

Prometheus + Grafana dashboards (latency, uploads, DB size, job queue).

Backup strategy: daily DB dump (pg_dump or WAL) + MinIO snapshot / object-store backups.

Rate limits, file type validation, antivirus integration (clamAV) as a background job.



Ops & deployment checklist (single-server)

Run app as systemd or Docker Compose service; expose via reverse proxy (Nginx/Caddy) with TLS.

MinIO data on separate disk mount; Postgres data on separate disk; rotate WAL daily.

Use a swap buffer if RAM constrained but monitor—swap hurts latency.

Backups: pg_basebackup + WAL or daily pg_dump + MinIO object snapshots saved off-server.

Monitoring: scrape app /metrics with Prometheus; deploy dashboards for queue depth, upload throughput, CPU, GC pauses.


How to split into microservices later (map for future you)

Candidate services to extract: attachments service (Go), search service (Elastic/Go), heavy analytics/reporting (Go/Java), mail ingestion (Go).

Communication: gRPC for internal services or message bus (Kafka or RabbitMQ). Keep the monolith API as the canonical entry point until you migrate.



---

Database schema (Postgres) — copy-paste SQL

This schema uses pgcrypto for UUIDs and tsvector search. It's opinionated but pragmatic for an enterprise ITSM MVP.

-- Enable UUID generation
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- ENUMS
CREATE TYPE ticket_status AS ENUM ('new','open','in_progress','resolved','closed','cancelled');
CREATE TYPE ticket_priority AS ENUM ('low','medium','high','critical');
CREATE TYPE ticket_type AS ENUM ('incident','request','change','problem','task');
CREATE TYPE change_status AS ENUM ('proposed','scheduled','in_progress','completed','rejected','cancelled');

-- Organizations
CREATE TABLE organizations (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text NOT NULL UNIQUE,
  parent_org uuid REFERENCES organizations(id),
  metadata jsonb DEFAULT '{}'::jsonb,
  created_at timestamptz DEFAULT now()
);

-- Users & Auth
CREATE TABLE users (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  username text UNIQUE,
  email text UNIQUE NOT NULL,
  display_name text,
  password_hash text, -- nullable if using SSO only
  auth_provider text DEFAULT 'local', -- local | oidc | saml | ldap
  external_id text, -- external provider id
  is_active boolean DEFAULT true,
  last_login_at timestamptz,
  metadata jsonb DEFAULT '{}'::jsonb,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

CREATE TABLE roles (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text NOT NULL UNIQUE,
  description text,
  created_at timestamptz DEFAULT now()
);

CREATE TABLE permissions (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text NOT NULL UNIQUE,
  description text
);

CREATE TABLE role_permissions (
  role_id uuid REFERENCES roles(id) ON DELETE CASCADE,
  permission_id uuid REFERENCES permissions(id) ON DELETE CASCADE,
  PRIMARY KEY (role_id, permission_id)
);

CREATE TABLE user_roles (
  user_id uuid REFERENCES users(id) ON DELETE CASCADE,
  role_id uuid REFERENCES roles(id) ON DELETE CASCADE,
  PRIMARY KEY (user_id, role_id)
);

-- Tickets
CREATE TABLE tickets (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  ticket_number bigserial NOT NULL UNIQUE, -- readable sequence
  organization_id uuid REFERENCES organizations(id),
  title text NOT NULL,
  description text,
  status ticket_status DEFAULT 'new',
  priority ticket_priority DEFAULT 'medium',
  type ticket_type DEFAULT 'incident',
  requester_id uuid REFERENCES users(id),
  assignee_id uuid REFERENCES users(id),
  created_by uuid REFERENCES users(id),
  updated_by uuid REFERENCES users(id),
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  closed_at timestamptz,
  metadata jsonb DEFAULT '{}'::jsonb,
  search_vector tsvector
);

CREATE INDEX idx_tickets_org_status ON tickets (organization_id, status);
CREATE INDEX idx_tickets_assignee ON tickets (assignee_id);
CREATE INDEX idx_tickets_created_at ON tickets (created_at);
CREATE INDEX idx_tickets_priority ON tickets (priority);
CREATE INDEX idx_tickets_search ON tickets USING GIN (search_vector);

-- Trigger to update tsvector on ticket changes
CREATE FUNCTION tickets_search_vector_update() RETURNS trigger AS $$
BEGIN
  NEW.search_vector :=
    to_tsvector('english', coalesce(NEW.title,'') || ' ' || coalesce(NEW.description,'') || ' ' || coalesce(NEW.metadata->>'keywords',''));
  RETURN NEW;
END
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_tickets_search_vector
BEFORE INSERT OR UPDATE ON tickets
FOR EACH ROW EXECUTE FUNCTION tickets_search_vector_update();

-- Ticket comments
CREATE TABLE ticket_comments (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  ticket_id uuid REFERENCES tickets(id) ON DELETE CASCADE,
  author_id uuid REFERENCES users(id),
  content text,
  is_public boolean DEFAULT true,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

CREATE INDEX idx_comments_ticket ON ticket_comments(ticket_id);

-- Attachments (metadata only; files live in MinIO)
CREATE TABLE attachments (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  object_key text NOT NULL,  -- s3 key / minio path
  filename text NOT NULL,
  content_type text,
  size bigint,
  checksum bytea,            -- sha256
  uploaded_by uuid REFERENCES users(id),
  created_at timestamptz DEFAULT now(),
  ref_count integer DEFAULT 0
);

CREATE INDEX idx_attachments_checksum ON attachments USING hash (checksum);

-- Mapping attachments to tickets / comments
CREATE TABLE ticket_attachments (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  ticket_id uuid REFERENCES tickets(id) ON DELETE CASCADE,
  attachment_id uuid REFERENCES attachments(id) ON DELETE CASCADE,
  comment_id uuid REFERENCES ticket_comments(id),
  added_by uuid REFERENCES users(id),
  created_at timestamptz DEFAULT now()
);

CREATE INDEX idx_ticket_attachments_ticket ON ticket_attachments(ticket_id);
CREATE INDEX idx_ticket_attachments_attachment ON ticket_attachments(attachment_id);

-- CMDB assets (basic)
CREATE TABLE assets (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid REFERENCES organizations(id),
  name text NOT NULL,
  asset_tag text,
  asset_type text,
  status text,
  owner_id uuid REFERENCES users(id),
  properties jsonb DEFAULT '{}'::jsonb,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

CREATE INDEX idx_assets_org ON assets(organization_id);

-- Change requests
CREATE TABLE changes (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  change_number bigserial NOT NULL UNIQUE,
  title text NOT NULL,
  description text,
  status change_status DEFAULT 'proposed',
  change_type text,
  requested_by uuid REFERENCES users(id),
  assigned_to uuid REFERENCES users(id),
  scheduled_start timestamptz,
  scheduled_end timestamptz,
  related_ticket_id uuid REFERENCES tickets(id),
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  metadata jsonb DEFAULT '{}'::jsonb
);

CREATE INDEX idx_changes_status ON changes(status);
CREATE INDEX idx_changes_assigned ON changes(assigned_to);

-- Knowledge base
CREATE TABLE kb_articles (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid REFERENCES organizations(id),
  title text NOT NULL,
  slug text UNIQUE,
  content text,
  summary text,
  author_id uuid REFERENCES users(id),
  is_published boolean DEFAULT false,
  published_at timestamptz,
  version integer DEFAULT 1,
  metadata jsonb DEFAULT '{}'::jsonb,
  search_vector tsvector,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

CREATE INDEX idx_kb_search ON kb_articles USING GIN (search_vector);

CREATE FUNCTION kb_search_vector_update() RETURNS trigger AS $$
BEGIN
  NEW.search_vector := to_tsvector('english', coalesce(NEW.title,'') || ' ' || coalesce(NEW.content,''));
  RETURN NEW;
END
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_kb_search_vector
BEFORE INSERT OR UPDATE ON kb_articles
FOR EACH ROW EXECUTE FUNCTION kb_search_vector_update();

-- Workflows & instances
CREATE TABLE workflows (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text NOT NULL,
  definition jsonb, -- state machine rules
  created_by uuid REFERENCES users(id),
  active boolean DEFAULT true,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

CREATE TABLE workflow_instances (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  workflow_id uuid REFERENCES workflows(id),
  entity_type text NOT NULL,
  entity_id uuid NOT NULL,
  current_state text,
  data jsonb DEFAULT '{}'::jsonb,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

-- Audit log
CREATE TABLE audit_logs (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  entity_type text NOT NULL,
  entity_id uuid,
  action text NOT NULL,
  actor_id uuid REFERENCES users(id),
  before_data jsonb,
  after_data jsonb,
  created_at timestamptz DEFAULT now()
);

CREATE INDEX idx_audit_entity ON audit_logs(entity_type, entity_id);

-- Notifications
CREATE TABLE notifications (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  recipient_id uuid REFERENCES users(id),
  channel text, -- email, websocket
  payload jsonb,
  is_read boolean DEFAULT false,
  sent_at timestamptz,
  created_at timestamptz DEFAULT now()
);

-- Inbound email processing
CREATE TABLE inbound_emails (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  message_id text UNIQUE,
  from_address text,
  to_address text,
  subject text,
  body text,
  headers jsonb,
  parsed_ticket_id uuid REFERENCES tickets(id),
  created_at timestamptz DEFAULT now()
);

-- Auth providers (SSO configs)
CREATE TABLE auth_providers (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text,
  provider_type text, -- oidc | saml | ldap
  config jsonb,
  created_at timestamptz DEFAULT now()
);

-- Refresh tokens
CREATE TABLE refresh_tokens (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES users(id) ON DELETE CASCADE,
  token text,
  expires_at timestamptz,
  created_at timestamptz DEFAULT now()
);

-- SLAs
CREATE TABLE slas (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid REFERENCES organizations(id),
  name text,
  conditions jsonb,
  target_response_interval interval,
  target_resolution_interval interval,
  created_at timestamptz DEFAULT now()
);

CREATE TABLE sla_violations (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  ticket_id uuid REFERENCES tickets(id),
  sla_id uuid REFERENCES slas(id),
  violated_at timestamptz,
  details jsonb
);

Practical DB notes & operational tips

No blobs in DB. Files live in MinIO; DB stores object_key + checksum + metadata. Do not deviate.

Deduplicate attachments by checksum; implement ref_count to garbage-collect unreferenced objects.

Search: keep tsvector updated via triggers OR update from app during writes (fewer triggers if you want simpler migrations).

Ticket numbers: ticket_number is bigserial global for MVP. If customers require per-tenant numbering, add a tenant_sequence table or per-tenant sequence later.

Partitioning: If attachments or audit logs grow huge, partition tables by created_at or organization_id. Add that later when needed.

Backups: daily pg_dump + incremental WAL shipping; MinIO snapshots and replication if possible.


Dev ergonomics & implementation hints

Use EF Core migrations (dotnet ef migrations) — keep schema and DDL in sync.

Implement repository/unit-of-work patterns to keep logic testable.

For uploads: use streaming endpoints with IFormFile streamed directly into MinIO with small buffer; do checksum streaming so you never materialize the whole file.

Use SignalR typed hubs and events like TicketUpdated, CommentAdded, NotificationCreated.

Use Hangfire if you want UI for background jobs; or use IHostedService + Redis queue for lower footprint.

Keep heavy CPU tasks (image processing, OCR, virus scan) in background workers or a Go microservice later.

Implement feature flags early for toggling experimental flows.


Go-to-market MVP features to prioritize (order of importance)

1. Tickets + attachments + SSO + audit logs


2. Search + notifications (SignalR)


3. CMDB + changes (basic)


4. KB + search + linking to tickets


5. Workflows + simple SLAs


6. Migration tool for BMC/Remedy exports



Final note (because I know you’ll ask)

This is a lean, pragmatic MVP architecture that keeps the heavy stuff out of the DB, gives you enterprise features fast (C# ecosystem), and a straightforward migration path to microservices (Go) when CPU/IO demands outgrow a single host. You have a deployable DB schema and a phase plan that produces real, sellable features at every release gate.

You’re not stuck with this forever — but follow modular boundaries in code (clear module packages, interfaces) so later extractions into Go microservices are surgical, not traumatic. Now get to the code; the world does not deserve another half-baked ITSM, but it absolutely will get one unless you move faster than the competition.

