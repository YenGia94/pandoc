# Lịch Hẹn Dịch Vụ — Software Requirements Specification (SRS)

| | |
|---|---|
| **Mã dự án** | `lich-hen-dich-vu` |
| **Phiên bản** | V1.1 |
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
| 2026-09-15 | Mục 6 (toàn bộ) | Cập nhật chuẩn cấu trúc màn hình | Điều chỉnh 3 màn hình theo 8 đầu mục cố định per skill pvh-spec mới: Mục đích quy trình / Sự kiện kích hoạt / Quy trình nghiệp vụ hệ thống / Mô tả bước / Mô tả yêu cầu / Các ràng buộc / Hành vi Action-Button / Thông tin bổ sung. Bổ sung 2 điểm cần xác nhận mới vào mục 8. | V1.1 |

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

### Màn hình SCR-01: Danh mục lịch hẹn *(Nhân viên dịch vụ — Web/App F-PF)*

**User Story:** US-03, US-04

#### 6.1.1 Mục đích quy trình
Hiển thị danh sách tất cả các lịch hẹn dịch vụ dành cho cửa hàng của nhân viên đang đăng nhập, giúp nhân viên tra cứu và nắm bắt lượng khách chuẩn bị tới. Đây là điểm truy cập chính vào toàn bộ luồng nghiệp vụ lịch hẹn trên app F-PF.

#### 6.1.2 Sự kiện kích hoạt
- Nhân viên dịch vụ chọn tab/menu **"Danh mục lịch hẹn"** trên ứng dụng F-PF.
- Màn hình tự động refresh khi có lịch hẹn mới đẩy từ CRM về `[CẦN XÁC NHẬN]`.

#### 6.1.3 Quy trình nghiệp vụ hệ thống

```
[Nhân viên mở màn hình]
        |
        v
[F-PF gọi API lấy danh sách lịch hẹn của cửa hàng]
        |
        +--[Thành công]--> [Hiển thị danh sách lịch hẹn]
        |                         |
        |                   [Nhân viên tìm kiếm / lọc]
        |                         |
        |                   [Danh sách được lọc hiển thị lại]
        |                         |
        |                   [Nhân viên nhấn vào 1 lịch hẹn]
        |                         |
        |                   [Chuyển sang màn hình Chi tiết lịch hẹn]
        |
        +--[Lỗi / Trống]---> [Hiển thị trạng thái Empty hoặc lỗi]
```

#### 6.1.4 Mô tả bước trong quy trình

| STT | Tên bước | Nhóm thực hiện | Đầu vào | Thực hiện | Đầu ra |
|---|---|---|---|---|---|
| 1 | Mở màn hình | Nhân viên dịch vụ | Phiên đăng nhập hợp lệ | Chọn tab "Danh mục lịch hẹn" | Màn hình hiển thị trạng thái loading |
| 2 | Tải danh sách | Hệ thống F-PF | Mã cửa hàng của nhân viên | Gọi API GET danh sách lịch hẹn theo cửa hàng | Danh sách lịch hẹn đổ ra màn hình |
| 3 | Tìm kiếm / Lọc | Nhân viên dịch vụ | Từ khoá (SĐT / Tên KH / Biển số) hoặc khoảng ngày | Nhập từ khoá hoặc chọn bộ lọc | Danh sách thu hẹp theo điều kiện |
| 4 | Xem chi tiết | Nhân viên dịch vụ | Lịch hẹn hiển thị trong danh sách | Nhấn vào 1 bản ghi | Chuyển sang màn hình Chi tiết lịch hẹn |

#### 6.1.5 Mô tả yêu cầu

- **REQ-03** *(Ubiquitous)*: Hệ thống SHALL hiển thị danh sách lịch hẹn thuộc cửa hàng của nhân viên đang đăng nhập, bao gồm các trường: Mã lịch hẹn, Tên khách hàng, Số điện thoại, Biển số xe, Ngày giờ hẹn, Trạng thái, Ưu tiên, Loại hình DV.
- **REQ-04** *(WHEN/SHALL)*: WHEN nhân viên nhập từ khoá vào ô tìm kiếm hoặc chọn khoảng ngày, hệ thống SHALL lọc và hiển thị lại danh sách lịch hẹn khớp với điều kiện đã chọn.

#### 6.1.6 Các ràng buộc

| Mã | Loại | Nội dung |
|---|---|---|
| NFR-01 | Phân quyền | Chỉ hiển thị lịch hẹn thuộc cửa hàng của nhân viên đang đăng nhập. Trả HTTP 403 nếu cố truy cập dữ liệu cửa hàng khác. |

#### 6.1.7 Hành vi Action / Button

| Action / Button | Điều kiện hiển thị | Hành vi khi click |
|---|---|---|
| Ô tìm kiếm | Luôn hiển thị | Cho phép nhập SĐT, Biển số hoặc Tên KH để lọc danh sách theo thời gian thực |
| Bộ lọc "Ngày hẹn" | Luôn hiển thị | Lọc danh sách lịch hẹn theo khoảng thời gian được chọn |
| Thẻ lịch hẹn (bất kỳ) | Luôn hiển thị | Chuyển sang màn hình Chi tiết lịch hẹn tương ứng |

#### 6.1.8 Trạng thái màn hình

| Trạng thái | Khi nào xảy ra | Hiển thị gì |
|---|---|---|
| Loading | Đang gọi API lấy danh sách | Skeleton UI các thẻ lịch hẹn |
| Có dữ liệu | API trả về ≥ 1 lịch hẹn | Danh sách các thẻ lịch hẹn |
| Empty | Không có lịch hẹn nào hoặc không tìm thấy kết quả | Icon trống + thông báo "Chưa có lịch hẹn nào" |
| Lỗi | Gọi API thất bại | Thông báo lỗi + nút Thử lại `[CẦN XÁC NHẬN]` |

---

### Màn hình SCR-02: Chi tiết lịch hẹn *(Nhân viên dịch vụ — Web/App F-PF)*

**User Story:** US-05, US-07

#### 6.2.1 Mục đích quy trình
Cho phép nhân viên dịch vụ xem toàn bộ thông tin chi tiết của một lịch hẹn cụ thể — bao gồm thông tin khách hàng, xe và nhu cầu dịch vụ — để chủ động chuẩn bị vật tư và nhân sự trước khi khách tới. Ngoài ra hỗ trợ thao tác dời lịch nếu khách có thay đổi.

#### 6.2.2 Sự kiện kích hoạt
- Nhân viên nhấn vào một bản ghi lịch hẹn trên màn hình Danh mục lịch hẹn (SCR-01).

#### 6.2.3 Quy trình nghiệp vụ hệ thống

```
[Nhân viên nhấn vào lịch hẹn ở SCR-01]
        |
        v
[F-PF gọi API GET chi tiết lịch hẹn theo ID]
        |
        +--[Thành công]--> [Hiển thị đầy đủ thông tin lịch hẹn]
        |                         |
        |                   [Nhân viên đọc thông tin]
        |                         |
        |          +---------------+---------------+
        |          |                               |
        |   [Không thao tác]              [Nhấn "Dời lịch"]
        |   [Nhấn "Quay lại"]                     |
        |          |                    [Popup chọn Ngày/Giờ mới]
        |          v                               |
        |   [Quay về SCR-01]           [Xác nhận --> F-PF gọi API cập nhật]
        |                                          |
        |                              [Đồng bộ về CRM --> Cập nhật lịch]
        |
        +--[Lỗi]----------> [Hiển thị thông báo lỗi]
```

#### 6.2.4 Mô tả bước trong quy trình

| STT | Tên bước | Nhóm thực hiện | Đầu vào | Thực hiện | Đầu ra |
|---|---|---|---|---|---|
| 1 | Mở chi tiết | Hệ thống F-PF | Mã lịch hẹn | Gọi API GET chi tiết theo ID | Hiển thị toàn bộ thông tin lịch hẹn |
| 2 | Đọc thông tin | Nhân viên dịch vụ | Dữ liệu hiển thị | Xem và ghi nhớ để chuẩn bị | *(Không có đầu ra hệ thống)* |
| 3 | Dời lịch (nếu có) | Nhân viên dịch vụ | Ngày/Giờ mới từ khách hàng | Nhấn "Dời lịch", chọn thời gian mới, xác nhận | F-PF cập nhật lịch hẹn và gọi API đồng bộ về CRM |
| 4 | Quay lại | Nhân viên dịch vụ | *(Không có)* | Nhấn "Quay lại" | Trở về màn hình Danh mục lịch hẹn (SCR-01) |

#### 6.2.5 Mô tả yêu cầu

- **REQ-05** *(Ubiquitous)*: Hệ thống SHALL hiển thị đầy đủ thông tin chi tiết của lịch hẹn gồm: Thông tin KH (Tên, SĐT, KTV yêu cầu), Thông tin xe (Phân loại, Loại/dòng, Hãng, Biển số, Ghi chú), Thông tin lịch (Ngày hẹn, Giờ hẹn, Loại hình DV, Nội dung yêu cầu), Phân loại (Kênh, Ưu tiên, Hạn xử lý).
- **REQ-09** *(WHEN/SHALL — Could)*: WHEN nhân viên nhấn "Dời lịch" và xác nhận ngày/giờ mới, hệ thống SHALL cập nhật thời gian hẹn và gọi API đồng bộ thông tin mới về CRM.

#### 6.2.6 Các ràng buộc

| Mã | Loại | Nội dung |
|---|---|---|
| NFR-01 | Phân quyền | Chỉ cho phép xem chi tiết lịch hẹn thuộc cửa hàng của nhân viên đang đăng nhập. |
| BR-REQ09 | Business Rule | Nút "Dời lịch" chỉ hiển thị khi trạng thái lịch hẹn là "Chờ nhận xử lý". `[CẦN XÁC NHẬN]` |

#### 6.2.7 Hành vi Action / Button

| Action / Button | Điều kiện hiển thị | Hành vi khi click |
|---|---|---|
| Nút "Dời lịch" | Chỉ khi trạng thái lịch hẹn là "Chờ nhận xử lý" | Mở popup chọn Ngày/Giờ mới. Sau khi xác nhận: F-PF cập nhật và đồng bộ về CRM. *(Chờ xác nhận nghiệp vụ — REQ-09)* |
| Nút "Quay lại" | Luôn hiển thị | Quay lại màn hình Danh mục lịch hẹn (SCR-01) |

#### 6.2.8 Danh sách trường dữ liệu

| Nhóm thông tin | Tên trường | Mô tả |
|---|---|---|
| **Thông tin KH** | Tên khách hàng | Tên đầy đủ khách hàng |
| **Thông tin KH** | Số điện thoại | SĐT liên lạc |
| **Thông tin KH** | KTV yêu cầu | Kỹ thuật viên mà khách yêu cầu phục vụ |
| **Thông tin xe** | Phân loại xe | Ví dụ: xe máy, ô tô |
| **Thông tin xe** | Loại/dòng xe | Ví dụ: Vision, SH, Civic |
| **Thông tin xe** | Hãng xe | Ví dụ: Honda, Toyota |
| **Thông tin xe** | Biển số xe | Biển số đăng ký |
| **Thông tin xe** | Ghi chú chung | Ghi chú liên quan đến xe |
| **Thông tin lịch** | Ngày hẹn | Ngày khách đặt hẹn đến |
| **Thông tin lịch** | Giờ hẹn | Giờ khách đặt hẹn đến |
| **Thông tin lịch** | Loại hình dịch vụ | Ví dụ: bảo dưỡng định kỳ, sửa chữa |
| **Thông tin lịch** | Nội dung yêu cầu | Mô tả chi tiết yêu cầu của khách |
| **Phân loại** | Kênh | Kênh tiếp nhận lịch hẹn (ví dụ: điện thoại, Zalo) |
| **Phân loại** | Ưu tiên | Mức độ ưu tiên của lịch hẹn |
| **Phân loại** | Hạn xử lý | Thời hạn cần hoàn thành dịch vụ |

---

### Màn hình SCR-03: Danh sách tiếp nhận *(Nhân viên tiếp nhận — Web/App F-PF)*

**User Story:** US-06, US-08, US-09, US-10

#### 6.3.1 Mục đích quy trình
Màn hình thực hiện nghiệp vụ tiếp đón khách thực tế tại trạm dịch vụ. Hệ thống tự động liên kết với Danh mục lịch hẹn thông qua 3 phương thức nhận diện (SĐT, QR biển số, Camera AI) để điền sẵn thông tin khách, giảm nhập liệu thủ công và tự động hoàn thành lịch hẹn sau khi tiếp nhận.

#### 6.3.2 Sự kiện kích hoạt
- Nhân viên tiếp nhận mở form tạo mới Tiếp nhận khi khách hàng tới trạm.
- Hệ thống Camera AI tự động nhận diện biển số xe khi khách vào cổng và khởi tạo draft form.

#### 6.3.3 Quy trình nghiệp vụ hệ thống

```
[Khách hàng tới trạm]
        |
        +--[Camera AI nhận diện]--> [Hệ thống tự khởi tạo draft form, fill dữ liệu]
        |
        +--[Nhân viên mở form mới]
                |
                +--[Nhập SĐT / Biển số]
                |        |
                |  [Hệ thống tìm trong Danh mục lịch hẹn "Chờ nhận xử lý"]
                |        |
                |  +--[Tìm thấy]---> [Tự động fill dữ liệu vào form]
                |  +--[Không thấy]--> [Form trống, nhân viên nhập thủ công]
                |
                +--[Quét QR biển số]
                         |
                   [Đọc mã QR -> tự động fill dữ liệu như phương thức SĐT]
                         |
        [Nhân viên xác nhận và hoàn chỉnh thông tin form]
                         |
                  [Nhấn "Tiếp nhận"]
                         |
        [F-PF lưu thông tin tiếp nhận]
                         |
        [F-PF tự động cập nhật lịch hẹn -> "Hoàn thành xử lý"]
                         |
        [F-PF gọi API đồng bộ kết quả về CRM]
```

#### 6.3.4 Mô tả bước trong quy trình

| STT | Tên bước | Nhóm thực hiện | Đầu vào | Thực hiện | Đầu ra |
|---|---|---|---|---|---|
| 1a | Nhận diện qua Camera AI | Hệ thống F-PF | Hình ảnh biển số từ camera | Nhận diện ký tự biển số, tra cứu Danh mục lịch hẹn "Chờ nhận xử lý" | Draft form được khởi tạo với dữ liệu fill sẵn |
| 1b | Nhận diện qua SĐT | Nhân viên tiếp nhận | SĐT do khách cung cấp | Nhập SĐT vào ô tìm kiếm, hệ thống tra cứu | Dữ liệu lịch hẹn tương ứng được fill vào form |
| 1c | Nhận diện qua QR biển số | Nhân viên tiếp nhận | Mã QR biển số | Nhấn nút quét QR, mở camera, quét mã | Biển số được đọc, hệ thống tra cứu và fill dữ liệu |
| 2 | Hoàn chỉnh thông tin | Nhân viên tiếp nhận | Dữ liệu đã fill + thông tin thực tế | Kiểm tra, bổ sung hoặc chỉnh sửa nếu cần | Form đầy đủ, sẵn sàng tiếp nhận |
| 3 | Xác nhận tiếp nhận | Nhân viên tiếp nhận | Form đã điền đầy đủ | Nhấn nút "Tiếp nhận" | F-PF lưu bản ghi tiếp nhận |
| 4 | Tự động hoàn thành lịch hẹn | Hệ thống F-PF | Bản ghi tiếp nhận vừa lưu | Tự cập nhật trạng thái lịch hẹn sang "Hoàn thành xử lý" | Lịch hẹn được đóng, gọi API đồng bộ về CRM |

#### 6.3.5 Mô tả yêu cầu

- **REQ-06** *(WHEN/SHALL)*: WHEN nhân viên nhập SĐT hoặc biển số, hoặc khi Camera AI nhận diện biển số, hệ thống SHALL tra cứu Danh mục lịch hẹn có trạng thái "Chờ nhận xử lý" và tự động fill dữ liệu tương ứng vào form Tiếp nhận.
- **REQ-07** *(WHEN/SHALL)*: WHEN nhân viên nhấn nút "Tiếp nhận" và form được lưu thành công, hệ thống SHALL tự động chuyển trạng thái lịch hẹn liên kết sang "Hoàn thành xử lý" và gọi API đồng bộ kết quả về CRM.

#### 6.3.6 Các ràng buộc

| Mã | Loại | Nội dung |
|---|---|---|
| BR-02 | Business Rule | Tính năng tự động fill dữ liệu (Camera AI / QR / SĐT) chỉ tìm kiếm trong Danh mục lịch hẹn có trạng thái "Chờ nhận xử lý". |
| NFR-01 | Phân quyền | Nhân viên chỉ được tạo Tiếp nhận thuộc cửa hàng mà nhân viên được phân quyền. |
| NFR-02 | Tích hợp | Khi gọi API đồng bộ về CRM thất bại, hệ thống ghi log và tự động retry tối thiểu 3 lần trong 15 phút. |

#### 6.3.7 Hành vi Action / Button

| Action / Button | Điều kiện hiển thị | Hành vi khi click |
|---|---|---|
| Ô nhập SĐT / Biển số | Luôn hiển thị trên form | Hệ thống tra cứu Danh mục lịch hẹn "Chờ nhận xử lý", tự fill dữ liệu nếu tìm thấy |
| Nút "Quét QR biển số" | Luôn hiển thị trên form | Mở camera thiết bị, đọc mã QR biển số, sau đó tra cứu và fill dữ liệu như khi nhập SĐT/Biển số |
| Nút "Tiếp nhận" | Khi form đã điền đủ thông tin bắt buộc | Lưu thông tin tiếp nhận; F-PF tự chuyển lịch hẹn sang "Hoàn thành xử lý" và đồng bộ về CRM |
| Camera AI | Tự động — background khi nhận diện được biển số | Khởi tạo draft form với dữ liệu fill sẵn, chờ nhân viên xác nhận |

#### 6.3.8 Thành phần tích hợp với Danh mục lịch hẹn

| Phương thức nhận diện | Input | Nguồn tra cứu | Kết quả khi khớp | Kết quả khi không khớp |
|---|---|---|---|---|
| Nhập SĐT | Số điện thoại khách hàng | Danh mục lịch hẹn — trạng thái "Chờ nhận xử lý" | Fill dữ liệu vào form tiếp nhận | Form trống, nhân viên nhập thủ công |
| Quét QR biển số | Biển số đọc từ QR | Danh mục lịch hẹn — trạng thái "Chờ nhận xử lý" | Fill dữ liệu vào form tiếp nhận | Form trống, nhân viên nhập thủ công |
| Camera AI | Biển số nhận diện từ camera | Danh mục lịch hẹn — trạng thái "Chờ nhận xử lý" | Khởi tạo draft form, fill sẵn dữ liệu | Không tạo draft, nhân viên tạo thủ công |

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
| 6.1.2 | Màn hình Danh mục lịch hẹn có tự động refresh khi có lịch hẹn mới từ CRM hay chỉ tải lại khi người dùng mở lại màn hình? | Dev / PM | ASAP |
| 6.2.6 (BR-REQ09) | Điều kiện ẩn/hiện nút "Dời lịch": chỉ ở trạng thái "Chờ nhận xử lý" hay còn trạng thái nào khác? | Nghiệp vụ | ASAP |
