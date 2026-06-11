# Architecture Decision Log / Open Questions

> File này liệt kê các hạng mục **KHÔNG được đưa thẳng vào folder structure** vì chúng là
> **quyết định kiến trúc lớn, phụ thuộc vào loại sản phẩm và roadmap thực tế** —
> không phải "thêm 1 folder cho an toàn".
>
> Mỗi mục cần được trả lời bằng 1 ADR (Architecture Decision Record) riêng,
> dựa trên dữ liệu/yêu cầu thực tế, **không phải dựa trên "có thể sau này cần"**.

---

## Cách dùng file này

1. Khi 1 mục dưới đây trở thành nhu cầu thực tế (có ticket, có metric, có yêu cầu sản phẩm) → tạo ADR trong `docs/adr/`.
2. ADR phải trả lời: **Context → Decision → Consequences**, đặc biệt là **chi phí triển khai vs lợi ích đo được**.
3. Không implement trước khi có ADR được approve — tránh "kiến trúc phòng hờ" (premature architecture) gây tốn effort không cần thiết và tăng độ phức tạp cho 100% feature để phục vụ <10% use case chưa chắc xảy ra.

---

## 1. Resilience Layer nâng cao

### Câu hỏi cần trả lời
- Tỷ lệ lỗi mạng/timeout hiện tại ở các thị trường mục tiêu là bao nhiêu? (cần số liệu thực, không phải ước đoán)
- Có API nào thực sự cần circuit breaker (tự động ngắt khi server lỗi liên tục) không, hay retry-with-backoff đơn giản đã đủ?
- Có cần queue request khi mất mạng (offline action queue) cho flow nào? (VD: tạo booking khi mất mạng → queue → gửi khi có mạng lại)

### Mặc định hiện tại (đã có, KHÔNG cần thêm gì)
- `core/network/interceptors/retry_interceptor.dart`: retry với exponential backoff cho lỗi 5xx/timeout.
- `core/network/interceptors/auth_interceptor.dart` + `refresh_token_interceptor.dart`: xử lý token expired.

### Khi nào cần ADR
- Khi đo được tỷ lệ lỗi mạng > X% ảnh hưởng đến UX thực tế (có dashboard/analytics chứng minh).
- Khi product yêu cầu rõ ràng: "user phải thao tác được khi mất mạng, sync khi có mạng lại" cho 1 flow cụ thể.

### Nếu cần — phạm vi tối thiểu
- KHÔNG cần `circuit_breaker.dart` riêng cho mobile — pattern này có giá trị cao nhất ở backend/gateway.
- Chỉ thêm `core/network/offline_queue/` cho **1-2 flow cụ thể** được product xác nhận, không làm generic cho toàn app.

---

## 2. Sync Engine / Offline-First

### ⚠️ Đây là quyết định kiến trúc LỚN NHẤT trong danh sách này

Offline-first với conflict resolution **ảnh hưởng trực tiếp đến cách model domain entity** (cần version vector/timestamp, cần soft-delete thay vì hard-delete, cần operation log) — **không thể** thêm vào sau khi đã code xong feature.

### Câu hỏi cần trả lời (PHẢI trả lời TRƯỚC khi code feature liên quan)
1. Sản phẩm có bắt buộc hoạt động đầy đủ khi offline không, hay chỉ cần "xem dữ liệu cũ" (read-only cache)?
2. Nếu cần ghi dữ liệu khi offline: conflict có thể xảy ra ở mức nào?
   - Single-user, single-device → conflict gần như không xảy ra → chỉ cần local cache + sync queue đơn giản.
   - Multi-device cùng user, hoặc multi-user cùng resource (VD: 2 admin cùng sửa 1 booking) → cần conflict resolution thật.
3. Team có ai có kinh nghiệm với CRDT/OT, hay cần research/training trước?
4. Timeline: feature nào cần offline-first **đầu tiên**? Có thể launch MVP **không có** offline-first và thêm sau cho riêng feature đó không?

### Mức độ phức tạp tham khảo
| Mức | Mô tả | Effort ước tính |
|---|---|---|
| Level 0 | Cache read-only (Hive/Drift cache response, hiển thị khi offline) | Đã có sẵn qua `infrastructure/cache/` |
| Level 1 | Queue write actions khi offline, replay khi online (single-user, last-write-wins) | 2-4 tuần / feature |
| Level 2 | Conflict detection (version field, hiển thị conflict cho user resolve) | 1-2 tháng / feature |
| Level 3 | Full CRDT/OT, auto-merge | 3-6+ tháng, thường cần research phase riêng |

### Khuyến nghị
- **KHÔNG** xây `core/sync/` generic cho toàn app từ đầu.
- Nếu MVP cần offline cho 1-2 flow cụ thể (VD: tạo booking offline) → implement **Level 1** trong `infrastructure/cache/` + `infrastructure/sync_queue/` của **riêng feature đó**.
- Generalize thành `core/sync/` **chỉ khi** ≥3 feature cần pattern giống nhau VÀ đã chứng minh Level 1 hoạt động tốt.

---

## 3. Plugin Architecture / Super App

### ⚠️ Đây là thay đổi LOẠI SẢN PHẨM, không phải thay đổi kiến trúc

"Super app với plugin marketplace, bật/tắt module runtime" (kiểu Grab/Gojek/Zalo) là **một category sản phẩm khác** so với "app booking/payment với nhiều feature" — đòi hỏi:
- Dynamic module loading (Flutter chưa có official support tốt, cần Flutter Add-to-App + native module hoặc embed JS/WASM runtime).
- Versioning riêng cho từng plugin, deploy độc lập.
- Sandbox/security model cho plugin của bên thứ 3 (nếu marketplace mở cho external dev).

### Câu hỏi cần trả lời
1. Roadmap công ty có **chính thức** xác nhận pivot sang super app trong 3-5 năm không? (Nếu chỉ là "có thể" → KHÔNG implement)
2. Nếu có: plugin do **nội bộ** phát triển (chỉ cần module hóa tốt, đã đạt được qua `features/` + Monorepo ở mục 8) hay cho **bên thứ 3** (cần sandbox/security riêng — phức tạp hơn nhiều)?
3. Có ngân sách/timeline riêng cho R&D dynamic module loading trên Flutter không? (Đây là vùng chưa mature, cần POC trước khi cam kết)

### Khuyến nghị
- Cấu trúc `features/` + Monorepo (Melos) ở v2 đã đủ cho **"module hóa nội bộ tốt"** — đây là 80% giá trị của "super app architecture" mà KHÔNG cần plugin marketplace.
- Plugin marketplace cho bên thứ 3 **chỉ nên bắt đầu** sau khi có ADR riêng + POC 1 module thử nghiệm thành công.
- KHÔNG tạo `features/plugin_marketplace/`, `plugin_loader/`, `plugin_registry/` cho đến khi ADR này được approve.

---

## 4. API Contract Validator (Runtime)

### Câu hỏi cần trả lời
- BE có publish OpenAPI/Swagger spec không?
- Hiện tại DTO/model được viết tay hay generate từ spec?

### Khuyến nghị
- Nếu BE có OpenAPI spec → dùng **build-time codegen** (`openapi_generator`, hoặc custom script generate Dart model + repository từ spec) thay vì runtime validator.
  - Lợi ích: lỗi contract được phát hiện ở **compile-time/CI**, không phải runtime trên device user.
- `core/api_contract/contract_validator.dart` (runtime) chỉ có ý nghĩa nếu:
  - BE không version API tốt, hoặc
  - Cần validate response từ third-party API không kiểm soát được (VD: payment gateway external).
- Trong trường hợp đó, đặt validator trong `infrastructure/` của **feature liên quan** (VD: `payment/infrastructure/validators/`), không phải `core/`.

---

## 5. Performance Layer riêng (memory/image optimizer, lazy loader)

### Câu hỏi cần trả lời
- Đã có metric cụ thể nào cho thấy vấn đề chưa? (VD: memory leak đo bằng DevTools, image load chậm đo bằng performance overlay)

### Khuyến nghị
- **KHÔNG** tạo `core/performance/` như 1 layer kiến trúc từ đầu — đây là **optimization**, nên làm theo data, không phải theo "best practice chung chung".
- Image optimization: dùng package có sẵn (`cached_network_image`, `flutter_image_compress`) ở nơi cần, không cần wrapper riêng trừ khi có >1 cách dùng khác nhau cần thống nhất.
- Lazy loading: áp dụng per-screen (`ListView.builder`, pagination ở `application/queries/`) — đã là practice chuẩn, không cần "layer".
- Khi có metric xấu cụ thể → tạo ADR cho giải pháp cụ thể đó (VD: "ADR-00XX: Image caching strategy cho feature Profile do memory tăng 40% sau khi mở 20 ảnh").

---

## Tổng kết nguyên tắc chung

> **"Architecture chuẩn bị cho khả năng (capability), không phải xây sẵn tính năng (feature) chưa có nhu cầu."**

Khi có đề xuất "thêm layer X cho enterprise-ready", luôn hỏi:

1. **Có data/yêu cầu cụ thể nào chứng minh cần ngay bây giờ không?**
2. **Chi phí maintain layer này cho 100% feature là bao nhiêu, so với lợi ích cho <20% use case?**
3. **Nếu thêm sau (khi thực sự cần) thì chi phí migrate có chấp nhận được không?**
   - Nếu CÓ (VD: feature flag, performance optimization) → để sau, không block hiện tại.
   - Nếu KHÔNG (VD: offline-first ảnh hưởng domain model) → cần quyết định SỚM nhưng bằng **nghiên cứu kỹ + ADR**, không phải bằng cách thêm folder rỗng "cho có".
