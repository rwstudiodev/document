viết doc theo cấu trúc sau: 
docs/
│
├── product/
│ ├── vision.md
│ ├── scope.md
│ ├── roadmap.md
│ ├── personas.md
│ ├── user-stories.md
│ ├── functional-requirements.md
│ ├── non-functional-requirements.md
│ └── business-rules.md
│
├── domain/
│ ├── domain-overview.md
│ ├── bounded-contexts.md
│ ├── aggregates.md
│ ├── entities.md
│ ├── value-objects.md
│ ├── workflows/
│ │ ├── booking-workflow.md
│ │ ├── payment-workflow.md
│ │ ├── driver-workflow.md
│ │ └── notification-workflow.md
│ └── state-machines/
│ ├── booking-status.md
│ └── payment-status.md
│
├── architecture/
│ ├── architecture-overview.md
│ ├── system-context.md
│ ├── container-diagram.md
│ ├── component-diagram.md
│ ├── deployment-diagram.md
│ ├── scalability.md
│ ├── caching-strategy.md
│ ├── error-handling.md
│ ├── observability.md
│ └── security-architecture.md
│
├── api/
│ ├── api-overview.md
│ ├── authentication.md
│ ├── conventions.md
│ ├── error-codes.md
│ ├── rest/
│ │ ├── auth.md
│ │ ├── booking.md
│ │ ├── payment.md
│ │ └── user.md
│ ├── websocket/
│ │ ├── channels.md
│ │ ├── events.md
│ │ └── payloads.md
│ └── postman/
│
├── database/
│ ├── erd.md
│ ├── schema.md
│ ├── indexes.md
│ ├── migrations.md
│ ├── partitioning.md
│ ├── backup-strategy.md
│ └── query-optimization.md
│
├── realtime/
│ ├── realtime-overview.md
│ ├── firebase-structure.md
│ ├── websocket-design.md
│ ├── event-catalog.md
│ ├── notification-flow.md
│ ├── sync-strategy.md
│ └── offline-strategy.md
│
├── mobile/
│ ├── architecture.md
│ ├── folder-structure.md
│ ├── state-management.md
│ ├── navigation.md
│ ├── dependency-injection.md
│ ├── localization.md
│ ├── themes.md
│ ├── build-flavors.md
│ ├── release-process.md
│ ├── coding-standards.md
│ └── modules/
│ ├── auth.md
│ ├── booking.md
│ ├── payment.md
│ └── profile.md
│
├── backend/
│ ├── architecture.md
│ ├── folder-structure.md
│ ├── services.md
│ ├── modules.md
│ ├── authentication.md
│ ├── authorization.md
│ ├── background-jobs.md
│ ├── queue-system.md
│ ├── cron-jobs.md
│ ├── coding-standards.md
│ └── integrations.md
│
├── infrastructure/
│ ├── environments.md
│ ├── docker.md
│ ├── kubernetes.md
│ ├── ci-cd.md
│ ├── deployment.md
│ ├── monitoring.md
│ ├── logging.md
│ ├── alerting.md
│ ├── secrets-management.md
│ ├── cloud-resources.md
│ └── disaster-recovery.md
│
├── adr/
│ ├── ADR-001-project-structure.md
│ ├── ADR-002-state-management.md
│ ├── ADR-003-database.md
│ ├── ADR-004-realtime.md
│ ├── ADR-005-authentication.md
│ └── ADR-template.md
│
├── runbooks/
│ ├── deployment.md
│ ├── rollback.md
│ ├── database-recovery.md
│ ├── firebase-issues.md
│ ├── websocket-issues.md
│ ├── payment-failures.md
│ ├── high-cpu.md
│ ├── high-memory.md
│ └── incident-management.md
│
└── diagrams/
├── architecture/
├── database/
├── sequence/
├── workflows/
├── deployment/
├── drawio/
├── mermaid/
└── excalidraw/


file readme có bổ sung Thứ tự đọc dành cho người mới vào dự án


```
product
 ↓
domain
 ↓
architecture
 ↓
database
 ↓
api
 ↓
realtime
 ↓
mobile
 ↓
infrastructure
 ↓
adr
 ↓
runbooks

```

Nguyên tắc 80/20 Thực tế 80% giá trị tài liệu nằm ở 8 file sau:

```


```


```
product/business-rules.md

domain/domain-overview.md

domain/workflows/booking-workflow.md

architecture/architecture-overview.md

database/erd.md

api/rest/*.md

realtime/event-catalog.md

adr/*


ok rồi. hãy viết lại doc

```
