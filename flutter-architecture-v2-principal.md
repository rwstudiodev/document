# Flutter Project Structure v2
## Feature-First + Contract Boundary + DDD Lite + Bloc + VSA + Monorepo-Ready

> Thiết kế cho kiến trúc nền tảng 3-5 năm, scale 20+ dev, có thể chuyển sang Monorepo (Melos) và Micro-Frontend (logical separation).

---

## 1. Cấu trúc tổng quan

```
lib/
│
├── app/
├── platform/
├── core/
├── shared/
├── features/
│
└── main.dart
```

---

## 2. App Layer

App layer chứa **App Context** — những thứ thuộc về toàn ứng dụng, không phải 1 feature riêng lẻ.

```
app/
│
├── bootstrap/
│   ├── bootstrap.dart
│   └── initialization/
│       ├── firebase_initializer.dart
│       ├── storage_initializer.dart
│       ├── logger_initializer.dart
│       └── crashlytics_initializer.dart
│
├── auth/                          # Auth là App Context, không phải Feature
│   ├── auth_bloc.dart
│   ├── auth_state.dart
│   ├── auth_event.dart
│   ├── auth_repository.dart
│   ├── current_user.dart
│   └── permission_manager.dart
│
├── state/
│   └── app_state_observer.dart
│
├── router/
│   ├── app_router.dart
│   ├── route_names.dart
│   └── guards/
│       └── auth_guard.dart
│
├── config/
│   ├── env/
│   │   ├── env_dev.dart
│   │   ├── env_staging.dart
│   │   ├── env_prod.dart
│   │   └── env_loader.dart
│   ├── app_config.dart
│   └── flavor.dart
│
├── services/
│   └── deep_link_service.dart
│
├── theme/
│   ├── app_theme.dart
│   ├── colors.dart
│   ├── typography.dart
│   └── spacing.dart
│
├── localization/
│   ├── l10n/
│   │   ├── app_en.arb
│   │   └── app_vi.arb
│   └── generated/
│
├── observers/
│   ├── bloc_observer.dart
│   └── route_observer.dart
│
├── di/
│   └── app_injection.dart
│
└── app.dart
```

> **Lý do `auth` nằm ở `app/`**: hầu hết feature đều phụ thuộc vào `currentUser`/`permission`. Nếu Auth là 1 feature ngang hàng, sẽ tạo dependency ngược (Booking → Auth feature) và phá vỡ Module Independence.

---

## 3. Platform Layer (MỚI)

Platform layer chứa **device capability abstraction** — không thuộc `core` (vì gắn với native API/device), không thuộc `features` (vì dùng chung nhiều nơi).

```
platform/
│
├── device/
│   └── device_info_service.dart
│
├── notification/
│   ├── push_notification_service.dart
│   └── local_notification_service.dart
│
├── biometrics/
│   └── biometric_service.dart
│
├── permissions/
│   └── permission_service.dart
│
├── location/
│   └── location_service.dart
│
├── camera/
│   └── camera_service.dart
│
└── storage/
    └── file_storage_service.dart
```

> **Ví dụ**: `CameraService` không thuộc `core` (vì `core` nên framework/business-agnostic ở mức network/db) và không thuộc `feature` (vì `profile`, `chat`, `kyc`... đều cần dùng camera).

---

## 4. Core Layer (Thu gọn)

Core chỉ chứa **infrastructure thuần túy, framework-agnostic, không có business logic**.

```
core/
│
├── network/
│   ├── client/
│   │   ├── api_client.dart
│   │   ├── dio_client.dart
│   │   └── graphql_client.dart
│   │
│   ├── interceptors/
│   │   ├── auth_interceptor.dart
│   │   ├── refresh_token_interceptor.dart
│   │   ├── logging_interceptor.dart
│   │   └── retry_interceptor.dart
│   │
│   └── models/
│       └── api_response.dart
│
├── database/
│   ├── local/
│   │   ├── hive/
│   │   ├── drift/
│   │   └── secure_storage/
│   ├── migrations/
│   └── database_manager.dart
│
├── security/
│   ├── token_manager.dart
│   ├── encryption_service.dart
│   ├── secure_storage.dart
│   ├── certificate_pinning.dart
│   └── jailbreak_detection.dart
│
├── analytics/
│   ├── analytics_service.dart
│   └── analytics_event.dart
│
├── logging/
│   ├── app_logger.dart
│   └── trace_id_provider.dart
│
├── event_bus/
│   ├── domain_event_bus.dart
│   ├── domain_events/
│   ├── integration_events/
│   └── system_events/
│
├── feature_flags/
│   ├── feature_flag_service.dart
│   ├── remote_config.dart
│   └── rollout_strategy.dart
│
├── errors/
│   ├── failures/
│   │   ├── failure.dart
│   │   ├── network_failure.dart
│   │   ├── server_failure.dart
│   │   └── cache_failure.dart
│   │
│   ├── exceptions/
│   │   ├── api_exception.dart
│   │   ├── network_exception.dart
│   │   └── unauthorized_exception.dart
│   │
│   └── handlers/
│       └── error_handler.dart
│
├── di/
│   └── core_injection.dart
│
└── abstractions/
    ├── repository.dart        # interface thuần, KHÔNG kèm implementation logic
    ├── usecase.dart
    ├── mapper.dart
    └── paginated_result.dart
```

> **Đã bỏ `core/base`** (`BaseBloc`, `BaseCubit`, `BasePage`).
> Lý do: sau 2-3 năm, base class dùng chung luôn bị thêm field/method để fix case riêng → trở thành **God Object**. Thay bằng **mixins/extensions** cho từng nhu cầu cụ thể (VD: `LoadingStateMixin`, `ErrorSnackbarMixin`).

> **Bổ sung mới (so với v2 ban đầu):**
> - `security/`: mở rộng đầy đủ — `token_manager`, `certificate_pinning`, `jailbreak_detection`. Đây là **must-have trước khi lên production** với app xử lý auth/payment, không phải "nice to have".
> - `logging/trace_id_provider.dart`: sinh `trace_id` duy nhất xuyên suốt 1 flow (login → payment → success), gắn vào mọi log/analytics event để debug production dễ hơn.
> - `event_bus/`: phân loại 3 nhóm event — `domain_events` (nội bộ 1 feature, VD `UserLoggedIn`), `integration_events` (xuyên feature, VD `PaymentCompleted`), `system_events` (hạ tầng, VD `AppResumed`, `ConnectivityChanged`). Giúp dev biết đặt event ở đâu và ai nên subscribe.
> - `feature_flags/`: mở rộng `remote_config` + `rollout_strategy` — cần thiết khi team >5 dev để tách deploy khỏi release (dark launch, A/B test).
>
> **Resilience (retry/timeout/fallback)**: **không** tạo `core/resilience/` riêng. Retry/timeout xử lý qua `core/network/interceptors/retry_interceptor.dart` (đã có). Circuit breaker và request queue **chỉ thêm khi có nhu cầu đo được** (VD: tỷ lệ lỗi mạng cao ở thị trường cụ thể) — xem `docs/architecture/decision-log.md`.

---

## 5. Shared Layer (Cấm Business Model)

Shared chỉ chứa thứ **thuần UI/utility, không mang ý nghĩa nghiệp vụ**.

```
shared/
│
├── design_system/
│   ├── atoms/
│   ├── molecules/
│   ├── organisms/
│   └── templates/
│
├── widgets/
│   ├── buttons/
│   ├── dialogs/
│   ├── app_bar/
│   ├── text_fields/
│   ├── loading/
│   └── empty/
│
├── extensions/
│   ├── context_extensions.dart
│   ├── string_extensions.dart
│   └── datetime_extensions.dart
│
├── constants/
│   └── app_constants.dart
│
├── utilities/
│   ├── validators/
│   ├── formatters/
│   └── helpers/
│
└── resources/
    ├── icons/
    ├── images/
    ├── animations/
    └── fonts/
```

> **❌ KHÔNG có `shared/models`**.
> `User`, `Booking`, `Payment` **không phải Shared** — chúng là domain model thuộc về 1 feature cụ thể. Đặt ở `shared` tạo implicit coupling: mọi feature đều "biết" shape của entity thuộc feature khác → nguồn gốc circular dependency.
> Nếu 1 entity thực sự cần dùng chéo feature → expose qua `contract/` của feature sở hữu nó.

---

## 6. Features Layer (Thay đổi lớn nhất)

### 6.1. Cấu trúc chuẩn mỗi feature

```
features/
│
├── auth/                  # (nếu có phần feature riêng ngoài app/auth, VD: register/forgot password UI)
├── profile/
├── booking/
├── payment/
├── notification/
└── settings/
```

Mỗi feature theo cấu trúc:

```
feature/
│
├── contract/
├── application/
├── infrastructure/
└── presentation/
```

---

### 6.2. `contract/` — Public API của feature (QUAN TRỌNG NHẤT)

```
booking/contract/
│
├── booking_api.dart          # Interface công khai — cách duy nhất feature khác tương tác
├── booking_models.dart        # DTO/model công khai (KHÔNG phải internal entity)
└── booking_events.dart        # Domain events công khai cho event bus
```

**Quy tắc:**
- ✅ Feature khác **chỉ được import** `contract/`.
- ❌ **Không được** import `infrastructure/repository_impl`, `application/handlers`, hay bất kỳ thứ gì ngoài `contract/`.

```dart
// ✅ ĐÚNG — Payment feature gọi Booking
import 'package:app/features/booking/contract/booking_api.dart';

// ❌ SAI — vi phạm boundary
import 'package:app/features/booking/infrastructure/repository_impl/booking_repository_impl.dart';
```

---

### 6.3. `application/` — DDD Lite (Domain + Orchestration)

```
booking/application/
│
├── entities/
│   └── booking_entity.dart
│
├── value_objects/
│   ├── booking_status.dart
│   └── booking_date_range.dart
│
├── aggregates/
│   └── booking_aggregate.dart
│
├── commands/
│   ├── create_booking_command.dart
│   └── cancel_booking_command.dart
│
├── queries/
│   ├── get_booking_query.dart
│   └── list_bookings_query.dart
│
├── handlers/
│   ├── create_booking_handler.dart
│   └── cancel_booking_handler.dart
│
├── workflows/
│   └── checkout_workflow.dart
│
├── policies/
│   └── cancellation_policy.dart
│
└── failures/
    └── booking_failure.dart
```

> **Không có folder `domain/` riêng.** `entities/value_objects/aggregates` được gộp vào `application/` cùng `commands/queries/handlers`.
>
> **Lý do**: 90% feature trong Flutter app không đủ phức tạp để tách riêng DDD Full layer — tách ra chỉ tạo thêm indirection không cần thiết.
>
> **⚠️ Ngoại lệ — Khi nào TÁCH `domain/` riêng:**
> Nếu feature thuộc nhóm **core business** (thường là `payment`, `booking`, `pricing`) và có:
> - Aggregate với nhiều invariants phức tạp (>5 business rules)
> - Nhiều `policies/` tương tác lẫn nhau
> - Domain logic được nhiều `handlers` khác nhau tái sử dụng
>
> → Tách thành:
> ```
> booking/application/
> ├── domain/              # entities, value_objects, aggregates, policies
> └── ...                  # commands, queries, handlers, workflows
> ```
> Quyết định này nên được ghi lại trong **ADR (Architecture Decision Record)** của feature đó.

---

### 6.4. `infrastructure/` — Thay cho Data Layer

```
booking/infrastructure/
│
├── datasource/
│   ├── remote/
│   │   └── booking_remote_datasource.dart
│   └── local/
│       └── booking_local_datasource.dart
│
├── repositories/
│   └── booking_repository_impl.dart   # implement interface từ application/ hoặc contract/
│
├── dto/
│   ├── request/
│   │   └── create_booking_request.dart
│   └── response/
│       └── booking_response.dart
│
├── mappers/
│   └── booking_mapper.dart
│
└── cache/
    └── booking_cache_manager.dart
```

> Đặt tên `infrastructure` thay vì `data` để **gần với ngôn ngữ Backend/DDD chuẩn** (Hexagonal Architecture: Domain - Application - Infrastructure) → tăng Full-stack Alignment.

---

### 6.5. `presentation/` — VSA hoàn toàn

```
booking/presentation/
│
├── booking_list/
│   ├── view.dart
│   ├── bloc.dart
│   ├── event.dart
│   ├── state.dart
│   ├── effect.dart
│   ├── mixins/
│   │   └── booking_list_listener_mixin.dart
│   └── widgets/
│       ├── booking_card.dart
│       └── booking_filter_bar.dart
│
├── booking_detail/
│   ├── view.dart
│   ├── bloc.dart
│   ├── event.dart
│   ├── state.dart
│   ├── effect.dart
│   └── widgets/
│
├── create_booking/
│   ├── view.dart
│   ├── bloc.dart
│   ├── event.dart
│   ├── state.dart
│   ├── effect.dart
│   └── widgets/
│
└── routes/
    └── booking_routes.dart
```

> **Lợi ích**: mọi thứ thuộc 1 màn hình nằm cùng 1 chỗ — `view`, `bloc`, `state`, `effect`, `widgets` con. Khi sửa/xóa 1 màn hình, không cần lục tìm rải rác ở `pages/`, `blocs/`, `widgets/`.

---

## 7. Ví dụ đầy đủ — Feature `booking`

```
features/booking/
│
├── contract/
│   ├── booking_api.dart
│   ├── booking_models.dart
│   └── booking_events.dart
│
├── application/
│   ├── entities/
│   │   └── booking_entity.dart
│   ├── value_objects/
│   │   ├── booking_status.dart
│   │   └── booking_date_range.dart
│   ├── aggregates/
│   │   └── booking_aggregate.dart
│   ├── commands/
│   │   ├── create_booking_command.dart
│   │   └── cancel_booking_command.dart
│   ├── queries/
│   │   ├── get_booking_query.dart
│   │   └── list_bookings_query.dart
│   ├── handlers/
│   │   ├── create_booking_handler.dart
│   │   └── cancel_booking_handler.dart
│   ├── workflows/
│   │   └── checkout_workflow.dart
│   ├── policies/
│   │   └── cancellation_policy.dart
│   └── failures/
│       └── booking_failure.dart
│
├── infrastructure/
│   ├── datasource/
│   │   ├── remote/
│   │   │   └── booking_remote_datasource.dart
│   │   └── local/
│   │       └── booking_local_datasource.dart
│   ├── repositories/
│   │   └── booking_repository_impl.dart
│   ├── dto/
│   │   ├── request/
│   │   │   └── create_booking_request.dart
│   │   └── response/
│   │       └── booking_response.dart
│   ├── mappers/
│   │   └── booking_mapper.dart
│   └── cache/
│       └── booking_cache_manager.dart
│
├── presentation/
│   ├── booking_list/
│   │   ├── view.dart
│   │   ├── bloc.dart
│   │   ├── event.dart
│   │   ├── state.dart
│   │   ├── effect.dart
│   │   └── widgets/
│   ├── booking_detail/
│   │   └── ...
│   ├── create_booking/
│   │   └── ...
│   └── routes/
│       └── booking_routes.dart
│
└── di/
    └── booking_module.dart
```

---

## 8. Monorepo Structure (Melos)

```
packages/
│
├── app/                  # Shell app — duy nhất phụ thuộc vào TẤT CẢ feature packages
│   ├── lib/
│   │   ├── app.dart
│   │   ├── router/
│   │   ├── di/
│   │   └── ...
│   └── pubspec.yaml
│
├── core/
├── platform/
├── design_system/
├── shared/
│
├── auth/                 # app-context, không phải feature package thường
├── profile/
├── booking/
├── payment/
├── notification/
└── settings/
```

### Quy tắc export package

Mỗi feature package **chỉ export `contract/`** trong `lib/<feature>.dart`:

```dart
// packages/booking/lib/booking.dart
export 'src/contract/booking_api.dart';
export 'src/contract/booking_models.dart';
export 'src/contract/booking_events.dart';

// KHÔNG export application/, infrastructure/
```

Cấu trúc bên trong package vẫn giữ `src/`:

```
packages/booking/
├── lib/
│   ├── booking.dart          # public export — CHỈ chứa contract
│   └── src/
│       ├── contract/
│       ├── application/
│       ├── infrastructure/
│       └── presentation/
└── pubspec.yaml
```

> Dart's `src/` convention: mọi thứ trong `lib/src/` không tự động export ra ngoài package trừ khi được `export` tường minh trong file barrel (`lib/booking.dart`). Đây chính là **enforcement tự nhiên** cho Contract Boundary ở cấp package.

---

## 9. Dependency Rules

### 9.1. Quy tắc tổng quát

```
Feature
   ↓
Shared / Platform
   ↓
Core
```

### 9.2. Bảng quy tắc cụ thể

| Từ | Đến | Cho phép? |
|---|---|---|
| `features/auth` | `core`, `shared`, `platform` | ✅ |
| `features/booking` | `features/payment/contract` | ✅ |
| `features/booking` | `features/payment/infrastructure` | ❌ |
| `features/booking` | `features/payment/application` | ❌ |
| `app/` | tất cả `features/*/contract` (để register routes/DI) | ✅ |
| `app/` | `features/*/infrastructure` trực tiếp | ❌ (qua DI module) |
| `shared/` | `features/*` (bất kỳ chiều nào) | ❌ |
| `core/` | `features/*`, `app/`, `platform/` | ❌ |
| `platform/` | `core/` | ✅ (platform có thể dùng core/errors, core/logging) |
| `platform/` | `features/*` | ❌ |

### 9.3. Cơ chế Enforcement (bắt buộc, không chỉ document)

Quy tắc trên **phải được enforce bằng tooling**, không chỉ dựa vào code review — vì với 20+ dev và deadline áp lực, vi phạm boundary là **gần như chắc chắn xảy ra** sau 6-12 tháng nếu không có gì chặn.

**a. Cấp Monorepo (Melos + pubspec dependencies)**
- Package `booking` chỉ declare dependency đến `payment` (không có cách nào import `payment/src/infrastructure` vì Dart `src/` không export ra ngoài).
- Đây là enforcement **mạnh nhất** vì compiler sẽ báo lỗi nếu vi phạm.

**b. Cấp single-package (chưa tách Monorepo ngay)**
- Dùng custom lint rule với package [`custom_lint`](https://pub.dev/packages/custom_lint) hoặc [`import_lint`](https://pub.dev/packages/import_lint):

```yaml
# import_lint.yaml
rules:
  - name: "no-cross-feature-internal-import"
    forbidden:
      - "package:app/features/*/application/**"
      - "package:app/features/*/infrastructure/**"
    exclude_self: true   # feature được import chính application/infrastructure của mình

  - name: "no-shared-to-feature"
    forbidden:
      - "package:app/features/**"
    target: "package:app/shared/**"

  - name: "no-core-to-feature"
    forbidden:
      - "package:app/features/**"
      - "package:app/app/**"
    target: "package:app/core/**"
```

**c. CI Gate**
- Chạy `import_lint` / `dependency_validator` trong CI pipeline, fail build nếu vi phạm.
- Định kỳ (VD: mỗi sprint) chạy dependency graph visualization (`dart pub deps --style=compact` hoặc tool như `flutter_architecture_analysis`) để phát hiện circular dependency sớm.

---

## 10. Architecture Governance

### 10.1. ADR (Architecture Decision Record)

Mỗi quyết định kiến trúc quan trọng (VD: "tách `domain/` riêng cho feature `payment`") cần ghi lại:

```
docs/adr/
├── 0001-feature-first-structure.md
├── 0002-contract-boundary-enforcement.md
├── 0003-payment-feature-separate-domain-layer.md
└── template.md
```

Template tối thiểu:
```markdown
# ADR-XXXX: <Tiêu đề>
## Status: Accepted / Proposed / Deprecated
## Context: Vấn đề gì đang gặp phải?
## Decision: Quyết định là gì?
## Consequences: Hệ quả (tốt và xấu)?
```

### 10.2. Roadmap Micro-Frontend

Cấu trúc này đạt **Logical Micro-Frontend** (independent development qua package boundary), chưa đạt **Physical Micro-Frontend** (independent deployment/runtime loading).

| Giai đoạn | Mục tiêu | Trạng thái với cấu trúc này |
|---|---|---|
| Giai đoạn 1 (0-12 tháng) | Logical separation, Contract Boundary | ✅ Đạt được ngay |
| Giai đoạn 2 (12-24 tháng) | Monorepo + Melos, CI/CD per-package | ✅ Đạt được với setup ở mục 8 |
| Giai đoạn 3 (24-36 tháng) | Physical Micro-Frontend (dynamic feature modules) | ⚠️ Cần đánh giá feasibility riêng — Flutter chưa có official dynamic module loading tốt như Android App Bundle |

> Quyết định theo đuổi Giai đoạn 3 hay dừng ở Giai đoạn 2 nên là 1 ADR riêng, đánh giá lại sau 18-24 tháng dựa trên nhu cầu thực tế (số lượng feature team, tần suất release riêng lẻ).

---

## 11. Test Structure

```
test/
│
├── unit/
│   └── features/
│       └── booking/
│           ├── application/
│           │   ├── handlers/
│           │   └── policies/
│           └── infrastructure/
│               ├── mappers/
│               └── repositories/
│
├── widget/
│   └── features/
│       └── booking/
│           └── presentation/
│               ├── booking_list/
│               └── create_booking/
│
├── integration/
│   └── flows/
│       └── checkout_flow_test.dart
│
└── golden/
    └── features/
        └── booking/
```

---

## 12. Tóm tắt thay đổi so với v1

| Hạng mục | v1 | v2 |
|---|---|---|
| Top-level folder | `modules/` | `features/` (Feature-First, theo VGV/Flutter Community) |
| Auth | Module ngang hàng | `app/auth` — App Context |
| Platform services | Lẫn trong `core/` | Tách riêng `platform/` |
| `core/base` | Có (BaseBloc/BaseCubit/BasePage) | **Bỏ** — thay bằng mixins |
| `shared/models` | Có | **Cấm** — domain model thuộc về feature |
| Domain layer | `domain/` riêng | Gộp vào `application/` (DDD Lite), tách riêng nếu feature phức tạp (có ADR) |
| Data layer | `data/` | `infrastructure/` (gần ngôn ngữ Backend hơn) |
| Module boundary | Import tự do giữa modules | `contract/` — chỉ public API được import |
| Presentation | `pages/`, `widgets/`, `blocs/` tách rời | VSA — gộp theo từng screen |
| Monorepo | Chưa chuẩn bị | Melos-ready, `src/` export pattern |
| Enforcement | Code review only | Lint rules + CI gate |
| Governance | Không có | ADR + Roadmap Micro-Frontend |

**Điểm đánh giá: 9.5/10** (10/10 khi có đủ Dependency Enforcement + Monorepo + ADR + Architecture Governance đang vận hành thực tế, không chỉ trên giấy).

---

## 13. Bổ sung v2.1 — Production Essentials

Sau khi rà soát thêm các yếu tố "enterprise must-have", 3 hạng mục sau **là gap thật sự** và đã được tích hợp trực tiếp vào cấu trúc ở trên (không tạo layer riêng tách biệt khỏi `core/`):

| Hạng mục | Vị trí | Khi nào cần |
|---|---|---|
| Multi-environment config (`dev/staging/prod`) | `app/config/env/` | Ngay từ ngày 1 — tránh hardcode URL/key theo build flavor |
| Security essentials (token manager, secure storage, cert pinning, jailbreak detection) | `core/security/` | Trước khi có user thật / trước khi tích hợp payment |
| Feature flags + remote config + rollout strategy | `core/feature_flags/` | Khi team >5 dev hoặc cần dark launch/A-B test |
| Event classification (domain/integration/system) | `core/event_bus/` | Khi có >3-4 feature cần giao tiếp qua event |
| Trace ID xuyên suốt flow | `core/logging/trace_id_provider.dart` | Khi có production traffic, cần debug theo flow |

### Những gì KHÔNG đưa vào structure (xem `decision-log.md`)

Các đề xuất sau **không phải thiếu sót của structure**, mà là **quyết định kiến trúc lớn, phụ thuộc loại sản phẩm** — không nên thêm "phòng hờ":

- Resilience layer nâng cao (circuit breaker, request queue, fallback strategy)
- Sync engine / Offline-first (conflict resolver, version control)
- Plugin architecture / Super app (plugin marketplace, dynamic module loading)
- API contract validator runtime riêng (nên dùng OpenAPI codegen build-time thay vì runtime validator)
- Performance layer riêng (memory/image optimizer) — optimize khi có metric đo được, không premature

→ Xem chi tiết và checklist quyết định tại **`decision-log.md`**.
