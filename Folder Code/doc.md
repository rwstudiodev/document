# System Complexity Levels (CRUD → Enterprise Platform)

---

# Level 1 — CRUD Application

## Đặc điểm
- Chủ yếu là CRUD (Create / Read / Update / Delete)
- Business logic rất đơn giản hoặc gần như không có
- Ít hoặc không có workflow nhiều bước
- Gần như không có integration bên ngoài
- Không cần queue, cache, search engine
- Scale chủ yếu là scale database + API stateless

## Ví dụ
- Admin CRUD lịch hẹn
- CMS nội dung
- Blog admin
- Todo / note app
- Inventory đơn giản cho nội bộ

## Thông thường
- ~1–3 domain chính
- < 20–30 API endpoints
- 1 database chính (MySQL/Postgres)
- Monolith backend
- Không cần Redis/Queue
- Dễ deploy, dễ maintain

---

# Level 2 — Business Application (Workflow-based App)

## Đặc điểm
- Có business logic rõ ràng (không còn chỉ CRUD)
- Xuất hiện workflow nhiều bước (state transition)
- Có validation nghiệp vụ phức tạp hơn
- Bắt đầu có transaction logic
- Có auth, role đơn giản
- Có notification, upload file, email
- Có thể có integration nhẹ (Firebase, email, payment đơn giản)

## Ví dụ
- App đặt lịch (spa, gym, clinic)
- Expense tracker (budget, recurring transaction)
- Booking hệ thống nhỏ
- CRM đơn giản cho SME
- Internal management system

## Thông thường
- 3–8 domain
- Workflow rõ (pending → confirmed → done…)
- Có transaction logic
- Redis (optional)
- Background job nhẹ (cron / queue đơn giản)
- Modular monolith bắt đầu xuất hiện
- ~30–80 API endpoints

---

# Level 3 — Business Platform (Multi-module System)

## Đặc điểm
- Nhiều domain liên kết chặt chẽ (multi-module platform)
- Business logic phức tạp, nhiều rule chồng nhau
- Workflow dài và nhiều bước async
- Bắt buộc có caching, queue, event-driven architecture
- Có integration hệ thống ngoài (payment, shipping, search, notification)
- Data consistency trở nên khó (eventual consistency xuất hiện)
- Có nhu cầu scale từng phần hệ thống

## Ví dụ
- Marketplace (Shopee-like)
- E-commerce (coupon, inventory, payment, shipping)
- Food delivery system
- Logistics tracking system
- Booking platform lớn

## Thông thường
- 8–20+ domain modules
- Message queue (Kafka / RabbitMQ)
- Redis cache layer
- Search engine (Elasticsearch)
- Object storage (S3)
- Background worker system
- Clean / Hexagonal / Modular monolith hoặc hybrid microservices
- Event-driven architecture rõ ràng

---

# Level 4 — Enterprise / Distributed Platform

## Đặc điểm
- Rất nhiều domain phức tạp, nhiều team phát triển song song
- Hệ thống phân quyền phức tạp (RBAC / ABAC)
- Audit log bắt buộc (compliance)
- Reporting / analytics nặng
- Multi-tenant SaaS hoặc hệ thống nội bộ quy mô lớn
- Integration với nhiều hệ thống (ERP, payment, legacy systems)
- High availability + fault tolerance là yêu cầu mặc định
- Distributed systems hoặc modular monolith cực kỳ kỷ luật

## Ví dụ
- CRM lớn (Salesforce-like)
- ERP (SAP-like)
- HRM enterprise
- Banking / fintech system
- Healthcare system cấp quốc gia
- SaaS platform multi-tenant

## Thông thường
- 20+ domain modules (có thể >100 bảng dữ liệu)
- Nhiều service hoặc bounded context rõ ràng
- Event-driven + async architecture mạnh
- Multi-layer caching + CDN + queue system
- Observability bắt buộc (logs, metrics, tracing)
- CI/CD + infrastructure automation đầy đủ
- Có governance giữa các team (architecture standards, ADRs)