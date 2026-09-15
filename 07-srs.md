# Hệ thống Quản lý Điều phối Bàn nâng (QMS) — Software Requirements Specification (SRS)

| | |
|---|---|
| **Mã dự án** | `QMS` |
| **Phiên bản** | V1.0 |
| **Ngày tạo** | 2026-09-15 |
| **Trạng thái** | Draft |

---

## Bảng ghi nhận thay đổi tài liệu

| Ngày | Vị trí thay đổi | Lý do | Mô tả thay đổi | Phiên bản mới |
|---|---|---|---|---|
| 2026-09-15 | — | Tạo mới | Phiên bản đầu tiên từ kết quả khảo sát nghiệp vụ | V1.0 |

---

## 1. Giới thiệu

### 1.1 Mục đích tài liệu

Tài liệu SRS này mô tả toàn bộ yêu cầu chức năng và phi chức năng của Hệ thống Quản lý Điều phối Bàn nâng (QMS), làm cơ sở thống nhất cho các bên UI/UX, Dev, QA và PM trong quá trình thiết kế, phát triển và kiểm thử. Tài liệu được tổ chức theo từng màn hình để UI đọc biết vẽ gì, Dev biết code gì, Test biết kiểm tra gì.

### 1.2 Phạm vi

**Goals — Làm trong phiên này:**
- Tự động hóa phân luồng xe vào đúng bàn nâng theo dịch vụ và thứ tự ưu tiên
- KTV thao tác nhận khách, gọi khách, tạm dừng, tiếp tục và hoàn tất sửa chữa trên Tablet
- Tivi hiển thị trạng thái bàn nâng và thống kê theo thời gian thực
- AI phát loa tự động khi match xe và khi KTV gọi khách tư vấn
- KT Trưởng / Cố vấn cấu hình bàn nâng trên **App và Web**

**Non-goals — Không làm trong phiên này:**
- App dành riêng cho khách hàng tự theo dõi hoặc đặt lịch từ xa
- Luồng Thanh toán — đã thiết kế ở phân hệ dịch vụ riêng
- KTV thao tác chuyển bàn nâng (chỉ KT Trưởng / Cố vấn)

### 1.3 Thuật ngữ & Từ viết tắt

| Thuật ngữ / Viết tắt | Định nghĩa |
|---|---|
| QMS | Queue Management System — Hệ thống quản lý hàng đợi và điều phối bàn nâng |
| KTV | Kỹ thuật viên — thực hiện sửa chữa tại bàn nâng |
| KT Trưởng | Kỹ thuật trưởng — quản lý kỹ thuật, có quyền cấu hình bàn nâng |
| F-PF | Nền tảng phần mềm dịch vụ đang dùng |
| Phiếu SC | Phiếu sửa chữa — đơn dịch vụ có sử dụng bàn nâng |
| Match | Hệ thống ghép một phiếu SC với một bàn nâng cụ thể |
| AI Phát loa | Text-to-Speech phát thông báo qua loa/âm ly |
| FIFO | First In First Out — phiếu vào trước được xử lý trước (trong cùng nhóm ưu tiên) |
| B1 / B2 / B3 | Màn hình chính / Màn hình chờ / Màn hình chi tiết phiếu trên Tablet KTV |

### 1.4 Tài liệu tham chiếu

| STT | Tên tài liệu | Mã / Nguồn |
|---|---|---|
| 1 | Proposal QMS | [01-proposal.md](./01-proposal.md) |
| 2 | User Stories QMS | [02-user-stories.md](./02-user-stories.md) |
| 3 | Requirements Spec QMS | [03-spec.md](./03-spec.md) |
| 4 | Test Scenarios QMS | [04-test-scenarios.md](./04-test-scenarios.md) |
| 5 | Biên bản khảo sát nghiệp vụ | Doccument/KhaoSatQMS_11.9.26.md |

---

## 2. Mô tả tổng quan

### 2.1 Bối cảnh & Vấn đề hiện tại

Quy trình điều phối xe vào bàn nâng đang hoàn toàn thủ công và rời rạc: KTV không biết xe nào tiếp theo, dữ liệu không liên thông giữa các bộ phận, và khách có lịch hẹn bị theo dõi ngoài hệ thống. Việc điều phối bằng miệng dẫn đến sai sót, mất thời gian và không có dấu vết trách nhiệm.

### 2.2 Đối tượng người dùng & Vai trò

| Vai trò | Thiết bị | Thao tác chính trên hệ thống |
|---|---|---|
| Kỹ thuật viên (KTV) | Tablet F-PF tại bàn nâng | Nhận khách, gọi khách, tạm dừng, tiếp tục, hoàn tất |
| Kỹ thuật trưởng / Cố vấn | **Tablet F-PF & Web F-PF** | Xem tổng quan bàn nâng, cấu hình bàn nâng |
| Tivi (màn hình hiển thị) | Kết nối F-PF | Hiển thị trạng thái bàn nâng realtime |

### 2.3 Giả định & Ràng buộc

**Giả định:**
- Phiếu sửa chữa có sử dụng bàn nâng đã được tạo đầy đủ từ phân hệ dịch vụ trước khi vào luồng QMS
- Mỗi bàn nâng đã được cấu hình sẵn danh sách dịch vụ được phép xử lý
- AI Text-to-Speech đã có sẵn hoặc sẽ được tích hợp; template text đọc loa đã được xác nhận nội bộ
- Loa/âm ly được kết nối vật lý với thiết bị trong garage

**Ràng buộc:**
- Phần cứng: Tablet F-PF cho KTV và KT Trưởng/Cố vấn; Tivi kết nối F-PF; Loa/âm ly kết nối máy tính
- Hệ thống hoạt động trong giờ làm việc của garage

### 2.4 Hệ thống liên quan / Phụ thuộc

| Hệ thống | Loại tích hợp | Mô tả |
|---|---|---|
| Phân hệ dịch vụ F-PF | Đọc | Nhận phiếu sửa chữa đã tạo, thông tin khách, lịch hẹn, loại dịch vụ |
| Phân hệ dịch vụ F-PF | Ghi | Cập nhật trạng thái hoàn tất sửa chữa để khởi động luồng thanh toán |
| AI Text-to-Speech | Gọi | Tạo và phát audio thông báo qua loa khi match xe hoặc gọi khách |

---

## 3. Luồng nghiệp vụ

### 3.1 Sơ đồ luồng tổng quan

```
[Phân hệ dịch vụ — Phiếu SC có sử dụng bàn nâng]
              │
              ▼
    ┌─────────────────────┐
    │   LUỒNG 1           │
    │   Validate phiếu    │
    │   ↔ năng lực bàn    │
    │   Xếp ưu tiên:      │
    │   Hẹn > VIP > Normal│
    └─────────┬───────────┘
              │
    ┌─────────▼───────────────────────────────────────┐
    │   LUỒNG 2 — ĐIỀU PHỐI & SỬA CHỮA               │
    │                                                 │
    │   KTV [Nhận khách]                              │
    │    ├─ Có xe → Match + Loa AI → B3 Phiếu SC     │
    │    └─ Chưa có xe → B2 Màn hình chờ             │
    │         └─ Khi có xe → Tự động match + Loa → B3│
    │                                                 │
    │   Tại B3:                                       │
    │    ├─ [Gọi khách] → Loa AI                     │
    │    ├─ [Tạm dừng] → Phiếu về DS (Tạm dừng)     │
    │    ├─ [Tiếp tục] → Ghi TG + Tivi cập nhật      │
    │    └─ [Hoàn tất] → Cập nhật phân hệ dịch vụ   │
    └─────────────────────────────────────────────────┘
```

### 3.2 Mô tả từng luồng

#### Luồng 1: Kiểm soát danh sách phiếu sửa chữa (Happy path)

1. Phân hệ dịch vụ tạo phiếu sửa chữa có sử dụng bàn nâng
2. Hệ thống QMS nhận phiếu và kiểm tra dịch vụ của phiếu với cấu hình từng bàn nâng
3. Phiếu được đưa vào hàng đợi của các bàn nâng phù hợp
4. Hệ thống sắp xếp hàng đợi theo ưu tiên: Lịch hẹn > VIP > Normal; cùng nhóm theo FIFO

**Exception:**
- Nếu phiếu không khớp bàn nâng nào → phiếu không vào hàng đợi, hệ thống cảnh báo [CẦN XÁC NHẬN — ISS-01]

#### Luồng 2: Điều phối & Sửa chữa (Happy path)

1. KTV bấm **[Nhận khách]** trên Tablet tại bàn nâng
2. Hệ thống match phiếu ưu tiên cao nhất phù hợp → phát loa AI thông báo xe vào bàn → hiển thị chi tiết phiếu SC trên Tablet → Tivi cập nhật
3. KTV thực hiện sửa chữa; có thể bấm **[Gọi khách]** để phát loa AI mời khách tư vấn
4. KTV bấm **[Hoàn tất]** → hệ thống cập nhật trạng thái hoàn tất sang phân hệ dịch vụ

**Luồng phụ — Chưa có khách:**
- Nếu hàng đợi trống → màn hình chờ khách; khi có xe phù hợp → tự động match và chuyển sang B3

**Luồng phụ — Tạm dừng:**
- KTV bấm **[Tạm dừng]** → phiếu về danh sách trạng thái Tạm dừng, giữ gắn KTV → bàn nâng nhận xe mới
- KTV bấm **[Tiếp tục]** → ghi nhận thời gian bắt đầu lại → Tivi cập nhật

### 3.3 User Stories

| Mã | User Story | Ưu tiên |
|---|---|---|
| US-01 | Hệ thống tự động validate và xếp hàng đợi phiếu sửa chữa theo ưu tiên và năng lực bàn nâng | Must |
| US-02 | KTV bấm Nhận khách và xem ngay thông tin phiếu sửa chữa trên Tablet | Must |
| US-03 | KTV bấm nút để hệ thống phát loa AI gọi khách đến tư vấn | Must |
| US-04 | KTV tạm dừng ca sửa chữa khi phát sinh sự cố hoặc chờ phụ tùng | Must |
| US-05 | KTV tiếp tục phiếu sửa chữa đang tạm dừng, kể cả từ hôm qua | Must |
| US-06 | Hệ thống tự động thông báo và chuyển màn hình khi có khách phù hợp | Must |
| US-07 | Tivi hiển thị trạng thái bàn nâng và thống kê theo thời gian thực | Must |
| US-08 | KT Trưởng / Cố vấn xem danh sách bàn nâng và cấu hình trên App & Web | Must |

---

## 4. Danh sách yêu cầu tổng hợp

| Mã | Tên yêu cầu | Loại | Màn hình liên quan | Ưu tiên |
|---|---|---|---|---|
| REQ-01 | Validate phiếu theo năng lực bàn nâng | Functional | — (backend) | Must |
| REQ-02 | Xếp hàng đợi phiếu theo thứ tự ưu tiên | Functional | — (backend) | Must |
| REQ-03 | Ưu tiên match theo KTV được chỉ định | Functional | — (backend) | Must |
| REQ-04 | KTV bấm Nhận khách để vào hàng đợi | Functional | B1 Màn hình chính | Must |
| REQ-05 | Tự động match và chuyển màn hình khi có khách | Functional | B2 Màn hình chờ | Must |
| REQ-06 | Phát loa AI khi match xe vào bàn nâng | Functional | — (hệ thống loa) | Must |
| REQ-07 | KTV gọi khách đến tư vấn qua loa | Functional | B3 Chi tiết phiếu | Must |
| REQ-08 | KTV tạm dừng sửa chữa | Functional | B3 Chi tiết phiếu | Must |
| REQ-09 | Phiếu tạm dừng hiển thị lại hôm sau | Functional | B1 Màn hình chính | Must |
| REQ-10 | KTV tiếp tục sửa chữa sau tạm dừng | Functional | B1 Màn hình chính | Must |
| REQ-11 | Tivi cập nhật khi KTV tiếp tục | Functional | Tivi | Must |
| REQ-13 | Tivi hiển thị danh sách bàn nâng realtime | Functional | Tivi | Must |
| REQ-14 | Tivi hiển thị thống kê tổng quan | Functional | Tivi | Must |
| REQ-15 | KT Trưởng xem tổng quan bàn nâng | Functional | C1 Tổng quan **(App & Web)** | Must |
| REQ-16 | Cấu hình thông tin bàn nâng | Functional | C2 Chi tiết **(App & Web)** | Must |
| NFR-01 | Độ trễ cập nhật Tivi ≤ 5 giây | Non-functional | Tivi | Must |
| NFR-02 | Độ trễ phát loa AI ≤ 3 giây | Non-functional | Hệ thống loa | Must |
| NFR-03 | Phân quyền | KTV chỉ thào tác bàn nâng được gán; KT Trưởng/Cố vấn có quyền xem và cấu hình tất cả | Toàn hệ thống | Must |
| NFR-04 | Thiết bị | Hoạt động trên Tablet F-PF (KTV); **App & Web F-PF (KT Trưởng/Cố vấn)**; Tivi kết nối F-PF | Toàn hệ thống | Must |
| NFR-05 | Audit | Ghi log: Nhận khách, Gọi khách, Tạm dừng, Tiếp tục, Hoàn tất — kèm ID người dùng, bàn nâng, phiếu, timestamp | Toàn hệ thống | Must |
| BR-01 | Chỉ phiếu có bàn nâng vào luồng QMS | Business Rule | — (backend) | — |
| BR-02 | Match đúng dịch vụ ↔ bàn nâng | Business Rule | — (backend) | — |
| BR-03 | Ưu tiên hàng đợi: Hẹn > VIP > Normal; FIFO cùng nhóm | Business Rule | — (backend) | — |
| BR-04 | Phiếu chỉ định KTV: chỉ match KTV đó | Business Rule | — (backend) | — |
| BR-05 | Tra SĐT/biển số → đổ danh sách lịch hẹn | Business Rule | Tiếp nhận | — |
| BR-06 | Tạm dừng giữ gắn KTV, hiển thị B1 hôm sau | Business Rule | B1, B3 | — |
| BR-07 | Loa AI khi: match xe và gọi khách tư vấn | Business Rule | Hệ thống loa | — |

---

## 5. Yêu cầu toàn hệ thống

### 5.1 Business Rules chung

| Mã | Quy tắc | Mô tả chi tiết |
|---|---|---|
| BR-01 | Chỉ phiếu có bàn nâng vào QMS | Phiếu sửa chữa phải được đánh dấu "có sử dụng bàn nâng" trong phân hệ dịch vụ mới được đẩy sang QMS |
| BR-02 | Match đúng dịch vụ ↔ bàn nâng | Phiếu chỉ được match vào bàn nâng có cấu hình dịch vụ tương ứng trong danh sách được phép |
| BR-03 | Thứ tự ưu tiên hàng đợi | Lịch hẹn > VIP > Normal; trong cùng nhóm sắp xếp theo thời gian vào danh sách (FIFO) |
| BR-04 | Phiếu chỉ định KTV | Nếu phiếu gắn tên KTV cụ thể, chỉ match vào bàn nâng của KTV đó |
| BR-05 | Nhận diện lịch hẹn | Khi NV Tiếp nhận tra SĐT hoặc quét biển số, hệ thống tự đổ danh sách lịch hẹn của khách |
| BR-06 | Phiếu Tạm dừng | Giữ nguyên gắn tên KTV; hiển thị trong danh sách B1 của bàn nâng đó kể cả ngày hôm sau |
| BR-07 | AI phát loa | Phát khi: (1) match xe vào bàn nâng, (2) KTV bấm [Gọi khách] |

### 5.2 Non-functional Requirements chung

| Mã | Nhóm | Yêu cầu | Ngưỡng đo được |
|---|---|---|---|
| NFR-01 | Hiệu năng | Cập nhật trạng thái lên Tivi sau mỗi thay đổi | ≤ 5 giây |
| NFR-02 | Hiệu năng | Bắt đầu phát loa AI sau khi kích hoạt | ≤ 3 giây |
| NFR-03 | Phân quyền | KTV chỉ thao tác bàn nâng được gán; KT Trưởng/Cố vấn có quyền xem và cấu hình tất cả | — |
| NFR-04 | Thiết bị | Hoạt động trên Tablet F-PF (KTV); **App & Web F-PF (KT Trưởng/Cố vấn)**; Tivi kết nối F-PF | — |
| NFR-05 | Audit | Ghi log: Nhận khách, Gọi khách, Tạm dừng, Tiếp tục, Hoàn tất — kèm ID người dùng, bàn nâng, phiếu, timestamp | — |

---

## 6. Chi tiết theo màn hình

---

### Màn hình B1: Màn hình chính bàn nâng *(KTV — Tablet)*

#### 6.1.1 Mục đích & User Story liên quan

Màn hình trung tâm của KTV tại bàn nâng. KTV dùng để nhận khách mới hoặc tiếp tục xe tạm dừng.

**User Story:** US-02, US-04, US-05

#### 6.1.2 Sự kiện kích hoạt

- KTV mở ứng dụng F-PF trên Tablet tại bàn nâng
- KTV bấm [Tạm dừng] từ màn hình B3

#### 6.1.3 Danh sách thành phần hiển thị

| Thành phần | Mô tả | Ghi chú |
|---|---|---|
| Danh sách phiếu SC | Danh sách xe đang phân công + xe tạm dừng gắn với bàn nâng này | Phiếu tạm dừng có nhãn "Tạm dừng" |
| Nút [Nhận khách] | Vào hàng đợi nhận khách mới | Luôn hiển thị |

#### 6.1.4 Hành vi action / button

| Action / Button | Điều kiện hiển thị | Hành vi khi click |
|---|---|---|
| **[Nhận khách]** | Luôn hiển thị | Có xe hợp lệ → Match + Loa AI → chuyển B3; Không có xe → chuyển B2 màn hình chờ |
| **Click phiếu Tạm dừng** | Khi có phiếu trạng thái Tạm dừng trong danh sách | Chuyển sang B3 chi tiết phiếu SC |
| **[Tiếp tục]** trên phiếu | Hiển thị trên dòng phiếu trạng thái Tạm dừng | Ghi nhận timestamp bắt đầu → chuyển B3 → Tivi cập nhật |

#### 6.1.5 Trạng thái màn hình

| Trạng thái | Khi nào xảy ra | Hiển thị gì |
|---|---|---|
| Danh sách rỗng | Không có phiếu nào phân công cho bàn | Thông báo "Không có phiếu nào" |
| Loading | Tải danh sách phiếu | Skeleton loader |

#### 6.1.6 Functional Requirements

**REQ-04: KTV bấm Nhận khách** *(Must)*
**WHEN** KTV bấm [Nhận khách], hệ thống **SHALL** kiểm tra hàng đợi và match nếu có phiếu, hoặc chuyển sang B2 nếu không có.

**REQ-09: Phiếu tạm dừng hiển thị lại hôm sau** *(Must)*
Hệ thống **SHALL** hiển thị phiếu trạng thái Tạm dừng gắn bàn nâng trong B1, kể cả phiếu từ ngày hôm trước.

**REQ-10: KTV tiếp tục sau tạm dừng** *(Must)*
**WHEN** KTV bấm [Tiếp tục] trên phiếu Tạm dừng, hệ thống **SHALL** ghi nhận timestamp và chuyển phiếu sang Đang xử lý.

#### 6.1.7 Business Rules áp dụng

- BR-06: Phiếu Tạm dừng giữ gắn KTV và hiển thị trong B1

---

### Màn hình B2: Màn hình chờ khách *(KTV — Tablet)*

#### 6.2.1 Mục đích & User Story liên quan

Hiển thị khi KTV bấm Nhận khách nhưng chưa có xe hợp lệ. Bàn nâng được xếp vào hàng đợi nhận khách.

**User Story:** US-06

#### 6.2.2 Sự kiện kích hoạt

- KTV bấm [Nhận khách] từ B1 và hàng đợi phiếu không có xe hợp lệ

#### 6.2.3 Hành vi action / button

| Action / Button | Điều kiện hiển thị | Hành vi khi click |
|---|---|---|
| **Tự động chuyển** | Khi có xe hợp lệ vào hàng | Hệ thống match + Loa AI + tự chuyển sang B3 (không cần KTV thao tác) |

#### 6.2.4 Trạng thái màn hình

| Trạng thái | Khi nào xảy ra | Hiển thị gì |
|---|---|---|
| Chờ | Bàn nâng trong hàng đợi nhận khách, chưa có xe | Thông báo đang chờ khách |

#### 6.2.5 Functional Requirements

**REQ-05: Tự động match và chuyển màn hình** *(Must)*
**WHEN** bàn nâng đang chờ và có phiếu hợp lệ, hệ thống **SHALL** tự động match, phát loa AI và chuyển Tablet sang B3.

---

### Màn hình B3: Chi tiết phiếu sửa chữa *(KTV — Tablet)*

#### 6.3.1 Mục đích & User Story liên quan

Màn hình làm việc chính trong quá trình sửa chữa. KTV xem thông tin phiếu và thực hiện các thao tác.

**User Story:** US-02, US-03, US-04, US-05

#### 6.3.2 Sự kiện kích hoạt

- Hệ thống auto-match thành công (từ B1 hoặc B2)
- KTV click vào phiếu Tạm dừng từ B1 và bấm [Tiếp tục]

#### 6.3.3 Danh sách thành phần hiển thị

| Thành phần | Mô tả | Ghi chú |
|---|---|---|
| Thông tin phiếu SC | Thông tin xe, khách, dịch vụ | Lấy từ phân hệ dịch vụ (Đã có) |
| Nút [Gọi khách] | Phát loa AI mời khách đến tư vấn | — |
| Nút [Tạm dừng] | Dừng sửa chữa, đẩy phiếu về danh sách | — |
| Nút [Hoàn tất] | Xác nhận hoàn thành ca sửa chữa | — |
| Thêm dịch vụ | Dùng nút "Thêm dịch vụ" từ module dịch vụ | Ghi chú nghiệp vụ — Đã có |

#### 6.3.4 Hành vi action / button

| Action / Button | Điều kiện hiển thị | Hành vi khi click |
|---|---|---|
| **[Gọi khách]** | Phiếu ở trạng thái Đang xử lý | Phát loa AI mời khách đến bàn nâng |
| **[Tạm dừng]** | Phiếu ở trạng thái Đang xử lý | Chuyển phiếu → Tạm dừng, giữ gắn KTV → về B1, Tivi xóa xe |
| **[Hoàn tất]** | Phiếu ở trạng thái Đang xử lý | Cập nhật trạng thái hoàn tất → phân hệ dịch vụ → về B1 |

#### 6.3.5 Functional Requirements

**REQ-07: KTV gọi khách qua loa** *(Must)*
**WHEN** KTV bấm [Gọi khách], hệ thống **SHALL** phát loa AI mời khách đến bàn nâng.

**REQ-08: KTV tạm dừng sửa chữa** *(Must)*
**WHEN** KTV bấm [Tạm dừng], hệ thống **SHALL** chuyển phiếu về danh sách trạng thái Tạm dừng, giữ nguyên gắn tên KTV.

#### 6.3.6 Business Rules áp dụng

- BR-06: Tạm dừng giữ gắn KTV
- BR-07: Loa AI phát khi Gọi khách

---

### Màn hình C1: Tổng quan bàn nâng *(KT Trưởng / Cố vấn — **Tablet & Web**)*

#### 6.4.1 Mục đích & User Story liên quan

Màn hình quản lý cho KT Trưởng và Cố vấn. Xem trạng thái toàn bộ bàn nâng và truy cập vào cấu hình từng bàn. Có thể truy cập trên cả **App F-PF (Tablet) lẫn Web F-PF**.

**User Story:** US-08

#### 6.4.2 Sự kiện kích hoạt

- KT Trưởng / Cố vấn mở ứng dụng F-PF trên Tablet hoặc trình duyệt Web

#### 6.4.3 Danh sách thành phần hiển thị

| Thành phần | Mô tả |
|---|---|
| Danh sách bàn nâng | Tất cả bàn nâng và trạng thái hiện tại |

#### 6.4.4 Hành vi action / button

| Action / Button | Điều kiện hiển thị | Hành vi khi click |
|---|---|---|
| **Click vào bàn nâng** | Luôn hoạt động | Chuyển sang C2 Chi tiết bàn nâng |

#### 6.4.5 Functional Requirements

**REQ-15: KT Trưởng xem tổng quan bàn nâng** *(Must)*
Hệ thống **SHALL** cung cấp màn hình Tổng quan bàn nâng trên **App và Web** với trạng thái tất cả bàn nâng; click vào bàn **SHALL** mở C2.

---

### Màn hình C2: Chi tiết bàn nâng *(KT Trưởng / Cố vấn — **Tablet & Web**)*

#### 6.5.1 Mục đích & User Story liên quan

Cấu hình thông tin bàn nâng: nhân viên đảm nhiệm và danh sách dịch vụ được phép thực hiện. Truy cập từ C1 trên **App hoặc Web**.

**User Story:** US-08

#### 6.5.2 Sự kiện kích hoạt

- KT Trưởng / Cố vấn click vào bàn nâng từ C1

#### 6.5.3 Danh sách trường dữ liệu

| Tên trường | Kiểu | Bắt buộc | Mô tả |
|---|---|---|---|
| Số bàn nâng | Text (read-only) | Có | Mã định danh bàn nâng |
| Nhân viên đảm nhiệm | Dropdown — danh sách KTV | Có | KTV được gán cho bàn |
| Danh sách dịch vụ | Multi-select — danh sách loại dịch vụ | Có | Dịch vụ bàn nâng được phép thực hiện |

#### 6.5.4 Hành vi action / button

| Action / Button | Điều kiện hiển thị | Hành vi khi click |
|---|---|---|
| **[Lưu]** | Luôn hiển thị | Lưu cấu hình; áp dụng từ lần match tiếp theo |
| **[Huỷ]** | Luôn hiển thị | Bỏ thay đổi, quay về C1 |

#### 6.5.5 Functional Requirements

**REQ-16: Cấu hình thông tin bàn nâng** *(Must)*
**WHEN** KT Trưởng cập nhật và lưu thông tin trên **App hoặc Web**, hệ thống **SHALL** lưu cấu hình gồm số bàn, nhân viên đảm nhiệm, và danh sách dịch vụ được phép.

#### 6.5.6 Business Rules áp dụng

- BR-02: Cấu hình dịch vụ ảnh hưởng trực tiếp đến việc match phiếu

---

### Màn hình Tivi: Hiển thị trạng thái bàn nâng

#### 6.6.1 Mục đích & User Story liên quan

Màn hình hiển thị chung cho toàn bộ bộ phận. Cập nhật realtime trạng thái các bàn nâng và thống kê trong ngày.

**User Story:** US-08

#### 6.6.2 Sự kiện kích hoạt

- Màn hình Tivi luôn hiển thị liên tục trong giờ làm việc; tự cập nhật khi có thay đổi trạng thái

#### 6.6.3 Danh sách thành phần hiển thị

**Khối 1 — Danh sách bàn nâng:**

| Cột | Nội dung |
|---|---|
| Số bàn nâng | Mã định danh |
| Biển số xe | Biển số xe đang xử lý tại bàn |
| Giờ bắt đầu | Thời điểm bắt đầu sửa chữa |
| Thời gian xử lý | Bộ đếm giờ realtime từ lúc bắt đầu |

**Khối 2 — Thống kê tổng quan:**

| Chỉ số | Nội dung |
|---|---|
| Tổng xe | Tổng số xe vào QMS trong ngày |
| Đã xử lý | Số xe đã hoàn tất |
| Đang chờ | Số phiếu trong hàng đợi chưa được match |
| Đang xử lý | Số xe đang được sửa tại bàn nâng |

**Khối 3:** Ngày giờ hiện tại.

#### 6.6.4 Functional Requirements

**REQ-13: Tivi hiển thị danh sách bàn nâng realtime** *(Must)*
Hệ thống **SHALL** hiển thị số bàn nâng, biển số xe, giờ bắt đầu và thời gian xử lý; cập nhật trong vòng 5 giây khi có thay đổi.

**REQ-14: Tivi hiển thị thống kê tổng quan** *(Must)*
Hệ thống **SHALL** hiển thị Tổng xe, Đã xử lý, Đang chờ, Đang xử lý và ngày giờ hiện tại.

#### 6.6.5 NFR đặc thù

- NFR-01: Cập nhật Tivi ≤ 5 giây sau mỗi thay đổi trạng thái

---

### Hệ thống loa AI (AI Text-to-Speech)

#### 6.7.1 Danh sách loại thông báo

| Mã | Sự kiện kích hoạt | Nội dung thông báo |
|---|---|---|
| NTF-01 | Hệ thống match xe vào bàn nâng | Thông báo biển số xe và số bàn nâng cần vào |
| NTF-02 | KTV bấm [Gọi khách] | Mời khách đến bàn nâng để tư vấn thêm |

#### 6.7.2 Chi tiết từng thông báo

**NTF-01: Thông báo xe vào bàn nâng**

| Mục | Nội dung |
|---|---|
| **Sự kiện kích hoạt** | Hệ thống thực hiện match phiếu SC với bàn nâng |
| **Đối tượng nhận** | Toàn bộ garage (qua loa/âm ly) |
| **Nội dung** | Có biển số xe và số bàn nâng (template đã được xác nhận nội bộ) |
| **Thời gian phát** | Trong vòng 3 giây sau khi match |

**NTF-02: Thông báo gọi khách tư vấn**

| Mục | Nội dung |
|---|---|
| **Sự kiện kích hoạt** | KTV bấm [Gọi khách] trên B3 |
| **Đối tượng nhận** | Toàn bộ garage (qua loa/âm ly) |
| **Nội dung** | Mời khách đến bàn nâng tư vấn thêm, có biển số xe và số bàn nâng |
| **Thời gian phát** | Trong vòng 3 giây sau khi bấm |

---

## 7. Đo lường thành công

| Chỉ số | Hiện tại | Mục tiêu | Cách đo |
|---|---|---|---|
| Điều phối bàn nâng bằng miệng | 100% | 0% | Tỷ lệ phiếu được match tự động bởi hệ thống |
| Thời gian Thu ngân biết xe hoàn tất | Phụ thuộc thông báo miệng | < 30 giây sau khi KTV bấm Hoàn tất | Log timestamp hệ thống |
| Ca sửa chữa có lưu vết QA của KTV | 0% | 100% | Số phiếu hoàn tất có gắn ID KTV và timestamp |
| Nhập liệu lại của Kế toán phụ tùng | Thủ công 100% | 0% | Số phiếu phụ tùng nhập tay sau khi triển khai |

---

## 8. Các điểm cần xác nhận

| Mục | Giả định hiện tại | Người cần xác nhận | Hạn |
|---|---|---|---|
| REQ-01.2 — Cơ chế cảnh báo khi phiếu không khớp bàn nâng nào | Hệ thống hiển thị cảnh báo cho người quản lý | Nghiệp vụ | — |
