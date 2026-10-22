# SignalOps Product Requirements

- Document ID: SIG-002
- Version: 1.0-draft
- Status: Proposed
- Primary persona: On-call SRE
- Product owner: Mahesh

Proposed product problem statement
SignalOps — AI-assisted incident intelligence

Engineering teams receive fragmented and repetitive operational alerts that make it harder to identify incidents, understand their impact, and find relevant troubleshooting procedures.

SignalOps will ingest alerts, group related alerts into incidents, maintain an auditable incident lifecycle, and use AI to recommend troubleshooting steps grounded in approved runbooks.

The platform will use asynchronous event processing to separate incident ingestion from AI analysis and notifications, with automated tests, observable failures, and reproducible deployment.

Primary persona: On-call SRE
Responsible for restoring service reliability during incidents


Goals
- Identify the important incident among many alerts.
- Understand its severity and current status.
- Find relevant troubleshooting instructions quickly.
- Maintain a record of what happened and what was recommended.
  Pain points
- Duplicate or noisy alerts.
- Manual switching between monitoring alerts and runbooks.
- Unclear ownership and incident status.
- Troubleshooting suggestions that aren't grounded in the system's documentation.
  In scope — first release
- REST API to ingest operational alerts.
- Incident creation and deduplication by a stable alert key.
- Incident status lifecycle: OPEN, ACKNOWLEDGED, RESOLVED.
- PostgreSQL persistence and database migrations.
- Kafka event-driven processing with reliable publication.
- Spring AI runbook retrieval and structured recommendations.
- A notification worker that records notification attempts.
- Input validation, error handling, health checks and automated tests.
- Docker Compose deployment, local Kubernetes manifests, and an AWS demo deployment plan.



Out of scope — first release
- Integrations with real PagerDuty, Datadog, Prometheus or Slack accounts.
- Automatic restarts or remediation of production infrastructure.
- Enterprise SSO, complex role-based access control and multi-tenancy.
- Multi-region high availability.
- A large frontend application, mobile app or elaborate dashboard.
- A managed Kubernetes cluster and managed Kafka deployment as mandatory sprint deliverables.
  US-01 — Ingest an alert
  P0

As an SRE, I want to submit an operational alert so that SignalOps can record it.

Acceptance criteria
- A valid alert receives a successful HTTP response containing an alert or incident identifier.
- The alert and its associated incident are persisted in PostgreSQL.
- A request missing required fields returns HTTP 400 with a useful error response.
- Repeating a request with the same idempotency key does not create duplicate business records.




US-02 — Group duplicate alerts
P0

As an SRE, I want matching alerts to share an incident so that alert noise is reduced.

Acceptance criteria
- Two alerts with the same service and deduplication key arriving while an incident is open attach to the same incident.
- Both alerts remain individually traceable.
- Alerts with different keys create separate incidents.
- Concurrent duplicate submissions do not create multiple open incidents for the same deduplication scope.




US-03 — Process events reliably
P0

As a platform engineer, I want incident events to be processed asynchronously without silently losing committed changes.

Acceptance criteria
- A committed incident change creates a durable event-publication record in the same database transaction.
- A background publisher sends pending events to Kafka and marks successful publications.
- If Kafka is temporarily unavailable, pending events remain available for retry.
- Reprocessing an event does not duplicate its business effect.




US-04 — Generate AI runbook recommendations
P0

As an SRE, I want troubleshooting guidance based on relevant runbooks so that I can investigate an incident faster.

Acceptance criteria
- The AI service retrieves relevant passages from a small, supplied runbook collection.
- The result contains a structured summary, possible causes, recommended steps, and references to retrieved runbooks.
- References must correspond to documents actually retrieved for the request.
- Invalid output or an unavailable AI provider is handled without silently marking the recommendation as successful.
- Recommendations are stored against the incident and remain advisory.




US-05 — Manage incident lifecycle
P0

As an on-call SRE, I want to acknowledge and resolve an incident so that its current state is clear.

Acceptance criteria
- An open incident can be acknowledged and subsequently resolved.
- An invalid state transition is rejected with an appropriate HTTP status.
- Each accepted transition records a timestamp.
- Resolved incidents are not silently reused as open incidents when a later alert arrives.




US-06 — Process notifications
P1

As an SRE, I want incident notifications to be generated so that important events are visible to the notification worker.

Acceptance criteria
- The notification worker consumes the appropriate incident events.
- Each notification attempt is logged with the incident identifier and event type.
- Repeated delivery does not create duplicate notification records for the same logical event.




US-07 — Operate and deploy the application
P0

As a developer, I want to start and inspect the application consistently so that I can develop, test, and demonstrate it.

Acceptance criteria
- The core stack starts from documented Docker Compose instructions.
- Services expose health information and useful logs.
- Automated tests run in CI for pull requests.
- Kubernetes manifests can deploy the services to a local cluster.
  - The AWS demo deployment has documented configuration, networking, and verification steps.