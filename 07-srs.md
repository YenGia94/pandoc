# Lịch Hẹn Dịch Vụ — Software Requirements Specification (SRS)

| | |
|---|---|
| **Mã dự án** | `lich-hen-dich-vu` |
| **Phiên bản** | V1.0 |
| **Ngày tạo** | 2026-09-14 |
| **Người lập** | Antigravity AI |
| **Người kiểm tra** | Phạm Thị Yên |
| **Người phê duyệt** | Phạm Thị Yên |
| **Trạng thái** | Draft |

---

## Bảng ghi nhận thay đổi tài liệu

| Ngày | Vị trí thay đổi | Lý do | Mô tả thay đổi | Phiên bản mới |
|---|---|---|---|---|
| 2026-09-14 | — | Tạo mới | Phiên bản đầu tiên tổng hợp từ 01, 02, 03 | V1.0 |

---

## 1. Giới thiệu

### 1.1 Mục đích tài liệu

Tài liệu SRS này mô tả toàn bộ yêu cầu chức năng và phi chức năng của phân hệ Lịch Hẹn Dịch Vụ, làm cơ sở thống nhất cho các bên: UI/UX, Dev, QA và PM trong quá trình thiết kế, phát triển và kiểm thử. Mục tiêu cốt lõi là số hóa và tự động hóa luồng tiếp nhận lịch hẹn từ CRM xuống trực tiếp ứng dụng cho nhân viên dịch vụ tại cửa hàng (F-PF).

### 1.2 Phạm vi

**Goals — Làm trong phiên này:**
- Nhân viên dịch vụ xem được toàn bộ Danh mục lịch hẹn của cửa hàng trực tiếp trên app F-PF mà không cần trao đổi qua kênh ngoài hệ thống.
- Hệ thống F-PF tự động chuyển lịch hẹn sang trạng thái Hoàn thành xử lý khi nhân viên cập nhật thành công ở màn hình Tiếp nhận, đồng thời tự động đồng bộ kết quả ngược về CRM.
- Hỗ trợ nghiệp vụ dời lịch `[ĐANG CHỜ KHÁCH HÀNG XÁC NHẬN]` và đồng bộ về CRM.
- Luồng dữ liệu lịch hẹn từ CRM đến F-PF được vận hành hoàn toàn trên hệ thống, có liên kết nội bộ liền mạch sang màn hình Tiếp nhận.

**Non-goals — Không làm trong phiên này:**
- Tạo mới lịch hẹn trực tiếp từ app F-PF — phân hệ Dịch vụ.
- Tạo Phiếu hẹn bán hàng trên app Sales Order (chuyển sang giai đoạn sau).
- Luồng tóm tắt cuộc gọi / video (luồng 3 trong tài liệu khảo sát).
- API Zalo OA (luồng 4 trong tài liệu khảo sát).
- Đồng bộ dữ liệu khách hàng / xe từ Data Warehouse lên CRM.

### 1.3 Thuật ngữ & Từ viết tắt

| Thuật ngữ / Viết tắt | Định nghĩa |
|---|---|
| F-PF | Ứng dụng/hệ thống quản lý nội bộ dành cho nhân viên tại cửa hàng. |
| Danh mục lịch hẹn | Bảng dữ liệu chứa các lịch hẹn dịch vụ do CRM đẩy sang hệ thống F-PF. |
| Danh sách tiếp nhận | Màn hình nghiệp vụ đón khách/xe vào trạm của hệ thống F-PF. |

### 1.4 Tài liệu tham chiếu

| STT | Tên tài liệu | Mã / Nguồn |
|---|---|---|
| 1 | Khảo Sát Lịch Hẹn - 10.9.26 | `KhaoSat_LichHen_10.9.26.md` |
| 2 | Proposal | `01-proposal.md` |
| 3 | User Stories | `02-user-stories.md` |
| 4 | Requirements | `03-spec.md` |

---

## 2. Mô tả tổng quan

### 2.1 Bối cảnh & vấn đề hiện tại

Hiện tại chưa có cơ chế hệ thống để truyền tải thông tin lịch hẹn từ CRM đến nhân viên dịch vụ tại cửa hàng. Khách hàng đặt lịch qua điện thoại, nhân viên CSKH tiếp nhận qua cuộc gọi rồi tự log thủ công lên CRM và theo dõi riêng lẻ. Cách vận hành này dẫn đến nguy cơ sót lịch, phản hồi chậm và không kiểm soát được trạng thái xử lý ở cấp độ hệ thống. Việc kết quả xử lý phải phản hồi ngược qua kênh ngoài làm tốn nguồn lực và dễ mất mát dữ liệu.

### 2.2 Đối tượng người dùng & vai trò

| Vai trò | Phòng ban | Việc họ làm trên hệ thống |
|---|---|---|
| Nhân viên dịch vụ | Cửa hàng | Xem Danh mục lịch hẹn của cửa hàng, xem chi tiết, và xử lý luồng tiếp nhận khách. |

### 2.3 Giả định & Ràng buộc

**Giả định:**
- CRM có khả năng gọi API GET/POST để đẩy dữ liệu Phiếu công việc, với loại phiếu là Phiếu hẹn và nhận kết quả từ F-PF.
- Trường "Phòng ban" trong Phiếu công việc được dùng để định tuyến dữ liệu về đúng phòng Dịch vụ.
- Một cửa hàng có thể có nhiều nhân viên dịch vụ, tất cả đều xem chung Danh mục lịch hẹn của cửa hàng.

**Ràng buộc:**
- CRM là hệ thống của bên thứ 3 — F-PF không can thiệp vào cấu trúc dữ liệu phía CRM.
- F-PF chịu trách nhiệm xây dựng toàn bộ API endpoint (GET + POST).
- Timeline dự kiến: 1–2 tuần.

### 2.4 Hệ thống liên quan / Phụ thuộc

| Hệ thống | Loại tích hợp | Mô tả |
|---|---|---|
| CRM (bên thứ 3) | Đọc / Ghi | CRM là nguồn dữ liệu đẩy Phiếu công việc hẹn sang F-PF và nhận kết quả xử lý cuối cùng từ F-PF. |

---

## 3. Luồng nghiệp vụ

### 3.1 Sơ đồ luồng tổng quan

1. **Hệ thống (CRM)** tạo Phiếu công việc loại Phiếu hẹn và gọi API đẩy dữ liệu sang **F-PF**.
2. **Hệ thống (F-PF)** nhận thông tin, đưa vào **Danh mục lịch hẹn** của cửa hàng dịch vụ tương ứng và tự động gán field `Type = Dịch vụ`.
3. **Nhân viên dịch vụ** mở app F-PF, xem Danh mục lịch hẹn và xem chi tiết để nắm thông tin trước khi khách tới.
4. Khi khách hàng tới, tại màn hình **Danh sách tiếp nhận**, nhân viên thao tác: Tìm số điện thoại / Quét QR biển số / hoặc thông qua Camera AI nhận diện.
5. **Hệ thống (F-PF)** lấy Danh mục lịch hẹn làm input, tự động truy xuất thông tin và đổ dữ liệu ra màn hình Tiếp nhận.
6. **Nhân viên dịch vụ** xử lý và cập nhật trạng thái thành **"Được tiếp nhận"** kèm các thông tin kết quả.
7. Sau khi tiếp nhận, **Hệ thống (F-PF)** tự động cập nhật trạng thái "Hoàn thành xử lý" cho lịch hẹn và gọi API đồng bộ về cho **CRM**.

**Luồng phụ / Ngoại lệ:**
- **Khách hàng thay đổi lịch (Dời lịch)**: Khách báo đổi lịch, nhân viên cập nhật dời lịch trên F-PF và đồng bộ về CRM.
- **Tìm kiếm / Lọc**: Nhân viên dịch vụ tìm kiếm trong Danh mục theo tên, biển số hoặc lọc theo ngày.

### 3.2 User Stories

| Mã | User Story | Ưu tiên |
|---|---|---|
| US-01 | Là hệ thống F-PF, tôi muốn nhận dữ liệu Phiếu hẹn từ CRM để tự động thêm vào Danh mục | Must |
| US-02 | Là hệ thống F-PF, tôi muốn trả kết quả xử lý về CRM để đồng bộ trạng thái cuối cùng | Must |
| US-03 | Là nhân viên dịch vụ, tôi muốn xem Danh mục lịch hẹn để biết khách sắp tới | Must |
| US-04 | Là nhân viên dịch vụ, tôi muốn lọc và tìm kiếm Danh mục lịch hẹn để tra cứu nhanh | Must |
| US-05 | Là nhân viên dịch vụ, tôi muốn xem chi tiết lịch hẹn để chuẩn bị vật tư | Must |
| US-06 | Là hệ thống F-PF, tôi muốn tự động hoàn thành lịch hẹn từ màn hình Tiếp nhận | Must |
| US-07 | Là nhân viên dịch vụ, tôi muốn dời lịch hẹn để đồng bộ thông tin nếu khách đổi ý | Could |
| US-08 | Là nhân viên tiếp nhận, tôi muốn tìm bằng SĐT để tự động đổ dữ liệu ra Tiếp nhận | Must |
| US-09 | Là nhân viên tiếp nhận, tôi muốn tìm bằng quét QR biển số để tự động đổ dữ liệu | Must |
| US-10 | Là hệ thống F-PF, tôi muốn nhận diện bằng Camera AI để tự động đổ dữ liệu Tiếp nhận | Must |

---

## 4. Danh sách yêu cầu nghiệp vụ tổng hợp

| Mã | Tên yêu cầu | Loại | Màn hình liên quan | Ưu tiên |
|---|---|---|---|---|
| REQ-01 | Nhận dữ liệu Phiếu hẹn từ CRM | Functional | Backend / API | Must |
| REQ-02 | Từ chối Phiếu công việc thiếu dữ liệu | Functional | Backend / API | Must |
| REQ-03 | Hiển thị Danh mục lịch hẹn | Functional | Danh mục lịch hẹn | Must |
| REQ-04 | Lọc và tìm kiếm lịch hẹn | Functional | Danh mục lịch hẹn | Must |
| REQ-05 | Xem chi tiết lịch hẹn | Functional | Chi tiết lịch hẹn | Must |
| REQ-06 | Tự động đổ dữ liệu ra màn hình Tiếp nhận | Functional | Danh sách tiếp nhận | Must |
| REQ-07 | Tự động hoàn thành lịch hẹn | Functional | Danh sách tiếp nhận | Must |
| REQ-08 | Đồng bộ kết quả xử lý về CRM | Functional | Backend / API | Must |
| REQ-09 | Dời lịch hẹn | Functional | Chi tiết lịch hẹn | Could |
| NFR-01 | Bảo mật và Phân quyền | Non-functional | Toàn hệ thống | Must |
| NFR-02 | Khả năng phục hồi tích hợp | Non-functional | Backend / API | Must |
| BR-01 | Định tuyến loại lịch hẹn | Business Rule | Backend / API | Must |
| BR-02 | Ràng buộc đầu vào Tiếp nhận | Business Rule | Danh sách tiếp nhận | Must |

---

## 5. Yêu cầu toàn hệ thống

### 5.1 Business Rules chung

| Mã | Quy tắc | Mô tả chi tiết |
|---|---|---|
| BR-01 | Định tuyến loại lịch hẹn | Mọi dữ liệu Phiếu công việc (với loại phiếu là Phiếu hẹn) đẩy từ CRM sang F-PF qua API này đều được hệ thống tự động gán mặc định Type = "Dịch vụ". |
| BR-02 | Ràng buộc đầu vào Tiếp nhận | Tính năng tự động đổ dữ liệu Tiếp nhận (Camera AI/QR/SĐT) chỉ tìm kiếm trong Danh mục lịch hẹn có trạng thái "Chờ nhận xử lý". |

### 5.2 Non-functional Requirements chung

| Mã | Nhóm | Yêu cầu | Ngưỡng đo được |
|---|---|---|---|
| NFR-01 | Phân quyền | Chỉ cho phép nhân viên thao tác trên Danh mục lịch hẹn của cửa hàng mà nhân viên đó được phân quyền. Trả về HTTP 403 nếu cố truy cập dữ liệu cửa hàng khác. | — |
| NFR-02 | Tích hợp | Khi có sự cố kết nối lúc đẩy kết quả về CRM, hệ thống ghi log và tự động retry tối thiểu 3 lần trong vòng 15 phút. | Tối đa 3 lần retry/15p |

---

## 6. Chi tiết theo màn hình

### Màn hình: Danh mục lịch hẹn

**Nền tảng:** Web Admin

#### 6.1.1 Mục đích & User Story liên quan
Hiển thị danh sách tất cả các lịch hẹn dịch vụ dành cho cửa hàng của nhân viên đang đăng nhập, giúp nhân viên tra cứu và nắm bắt lượng khách chuẩn bị tới.
**User Story:** US-03, US-04

#### 6.1.2 Sự kiện kích hoạt
- Người dùng chọn tab/menu "Danh mục lịch hẹn" trên ứng dụng F-PF.

#### 6.1.3 Danh sách trường dữ liệu

| Tên trường (Cột/Card) | Kiểu | Bắt buộc | Mặc định | Validation rule |
|---|---|---|---|---|
| Mã lịch hẹn | Text | _ | — | |
| Tên khách hàng | Text | _ | — | |
| Số điện thoại | Text | _ | — | |
| Biển số xe | Text | _ | — | |
| Ngày giờ hẹn | Datetime | _ | — | |
| Trạng thái | Badge | _ | Chờ nhận xử lý | "Chờ nhận xử lý", "Hoàn thành xử lý" |
| Ưu tiên | Icon | _ | — | Hiển thị icon tương ứng nếu là ưu tiên |
| Loại hình DV | Text | _ | — | |
| Cửa hàng | Text | _ | — | |

#### 6.1.4 Hành vi action / button

| Action / Button | Điều kiện hiển thị | Hành vi khi click |
|---|---|---|
| Nút "Tìm kiếm" | Luôn hiển thị | Cho phép nhập SĐT, Biển số hoặc Tên KH để lọc danh sách |
| Bộ lọc "Ngày hẹn" | Luôn hiển thị | Lọc danh sách lịch hẹn theo khoảng thời gian được chọn |
| Bấm vào 1 Lịch hẹn | Luôn hiển thị | Chuyển sang màn hình Chi tiết lịch hẹn tương ứng |

#### 6.1.5 Trạng thái màn hình

| Trạng thái | Khi nào xảy ra | Hiển thị gì |
|---|---|---|
| Loading | Đang gọi API lấy danh sách | Skeleton UI các thẻ lịch hẹn |
| Empty | Không có lịch hẹn nào / Không tìm thấy | Icon trống + thông báo "Chưa có lịch hẹn nào" |

#### 6.1.6 Functional Requirements
- **REQ-03: Hiển thị Danh mục lịch hẹn**
- **REQ-04: Lọc và tìm kiếm lịch hẹn**

---

### Màn hình: Chi tiết lịch hẹn

#### 6.2.1 Mục đích & User Story liên quan
Xem toàn bộ thông tin chi tiết mà khách hàng và CSKH đã ghi chú để chuẩn bị vật tư, nhân sự phục vụ.
**User Story:** US-05, US-07

#### 6.2.2 Sự kiện kích hoạt
- Người dùng nhấn vào một bản ghi cụ thể trên màn hình Danh mục lịch hẹn.

#### 6.2.3 Danh sách trường dữ liệu

| Nhóm thông tin | Tên trường |
|---|---|
| **Thông tin KH** | Tên khách hàng, Số điện thoại, KTV yêu cầu |
| **Thông tin xe** | Phân loại xe, Loại/dòng xe, Hãng xe, Biển số xe, Ghi chú chung |
| **Thông tin lịch** | Ngày hẹn, Giờ hẹn, Loại hình dịch vụ, Nội dung yêu cầu |
| **Phân loại** | Kênh, Ưu tiên, Hạn xử lý |

#### 6.2.4 Hành vi action / button

| Action / Button | Điều kiện hiển thị | Hành vi khi click |
|---|---|---|
| Nút "Dời lịch" | Khi trạng thái là "Chờ nhận xử lý" | Mở popup chọn Ngày/Giờ mới. *(Chờ xác nhận nghiệp vụ)* |
| Nút "Quay lại" | Luôn hiển thị | Quay lại Danh mục lịch hẹn |

#### 6.2.5 Functional Requirements
- **REQ-05: Xem chi tiết lịch hẹn**
- **REQ-09: Dời lịch hẹn** (Could)

---

### Màn hình: Danh sách tiếp nhận

#### 6.3.1 Mục đích & User Story liên quan
Màn hình thực hiện nghiệp vụ tiếp đón khách thực tế tại trạm. Có khả năng tự động liên kết với Danh mục lịch hẹn để điền sẵn thông tin.
**User Story:** US-06, US-08, US-09, US-10

#### 6.3.2 Sự kiện kích hoạt
- Khách hàng tới trạm, nhân viên mở form tạo mới Tiếp nhận.
- Hệ thống AI tự quét biển số xe khi khách vào cổng.

#### 6.3.3 Hành vi action / button (Phần tích hợp)

| Action / Button | Điều kiện hiển thị | Hành vi khi click |
|---|---|---|
| Nhập SĐT / Biển số | Nhập trên ô tìm kiếm form | Hệ thống tự quét trong Danh mục lịch hẹn "Chờ nhận xử lý", tự fill dữ liệu. |
| Quét mã QR biển số | Nút trên form | Mở camera, quét mã -> tự động fill dữ liệu như trên. |
| Camera AI | Tự động (background) | Nhận diện -> tự tạo draft form với thông tin đã fill sẵn. |
| Nút "Tiếp nhận" | Khi điền đủ thông tin form | Lưu thông tin tiếp nhận. F-PF tự chuyển lịch hẹn sang "Hoàn thành xử lý". |

#### 6.3.4 Functional Requirements
- **REQ-06: Tự động đổ dữ liệu ra màn hình Tiếp nhận**
- **REQ-07: Tự động hoàn thành lịch hẹn**
- **BR-02: Ràng buộc đầu vào Tiếp nhận** (Chỉ quét lịch hẹn chưa xử lý)

---

## 7. Đo lường thành công

| Chỉ số | Hiện tại | Mục tiêu | Cách đo |
|---|---|---|---|
| Tỷ lệ lịch hẹn bị sót / xử lý muộn | Không có baseline | Giảm đáng kể | So sánh báo cáo CRM trước và sau go-live |
| Thời gian NV nhận được thông tin lịch hẹn | Không đo được (kênh ngoài)| Tức thì sau khi CRM tạo phiếu | Log timestamp API |
| Tỷ lệ kết quả xử lý cập nhật lên hệ thống | Không có baseline | ≥ 95% | Báo cáo tỷ lệ khớp dữ liệu CRM / F-PF |

---

## 8. Các điểm cần xác nhận

| Mục | Giả định hiện tại | Người cần xác nhận | Hạn |
|---|---|---|---|
| 3.1 & 6.2 (REQ-09) | Nghiệp vụ "Dời lịch hẹn" có được đưa vào phạm vi (Scope) phát triển lần này hay không? | PM / Nghiệp vụ | ASAP |
