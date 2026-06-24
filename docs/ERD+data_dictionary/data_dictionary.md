# Mini-Wallet Data Dictionary

## 1. Overview

Data Dictionary mô tả các model chính trong hệ thống Mini-Wallet, bao gồm field, kiểu dữ liệu, khóa, trạng thái bắt buộc và ý nghĩa sử dụng.

Các model được chia thành 4 nhóm:

- User / Wallet
- Configuration Engine
- Transaction Runtime
- Bill Payment

## 2. User
**Mục đích: ** Lưu thông tin người dùng trong hệ thống

| Field | Type | Key | Required | Description |
|---|---|---|---|---|
|id|String/Object|PK|Yes|Khóa chính của user|
|phone|String|Unique|Yes|Số điện thoại dùng để đăng nhập và giao dịch|
|fullName|String| |Yes|Họ và trên người dùng|
|pinHash|String| |No|Pin đã hash|
|role|String| |Yes|Vai trò: customer/officer/admin|
|state|String|  |Yes|Trạng thái runtime: normal/inProgress|
|status|String|  |Yes|Trạng thái user: active/inactive/locked|
|createdAt|Date |  |Yes|Thời gian tạo|
|updatedAt|Date|  |Yes|Thời gian cập nhật|

---
## 3. Pocket

**Purpose:** Lưu ví và số dư hiện tại.

| Field | Type | Key | Required | Description |
|---|---|---|---|---|
| id | String/ObjectId | PK | Yes | Khóa chính của ví |
| user | String/ObjectId | FK | No | Tham chiếu đến User.id, null nếu là ví system/bank |
| client | String |  | Yes | Loại ví: customer/biller/system/bank |
| currency | String |  | Yes | Loại tiền, ví dụ MMK |
| balance | Number |  | Yes | Số dư hiện tại |
| checksum | String |  | Yes | Mã kiểm tra chống sửa balance thủ công |
| status | String |  | Yes | active/inactive/locked |
| createdAt | Date |  | Yes | Thời gian tạo |
| updatedAt | Date |  | Yes | Thời gian cập nhật |

---
## 4. Service

**Purpose:** Lưu cấu hình nghiệp vụ như P2P, Cash-in, Bill Payment.

| Field | Type | Key | Required | Description |
|---|---|---|---|---|
| id | String/ObjectId | PK | Yes | Khóa chính service |
| code | String | Unique | Yes | Mã nghiệp vụ: P2P/CASH_IN/BILL_PAYMENT |
| name | String |  | Yes | Tên nghiệp vụ |
| fieldBuilder | Array |  | Yes | Danh sách luật dựng biến TRANSBODY |
| amountFormula | Object/String |  | No | Công thức tính amount nếu cần |
| action | String |  | Yes | none hoặc billerTrans |
| actionParams | Object |  | No | Tham số action, ví dụ billerId |
| fee | Object |  | Yes | Cấu hình phí |
| auth | Object |  | Yes | Phương thức xác thực: PIN/NONE |
| status | String |  | Yes | active/inactive |
| createdAt | Date |  | Yes | Thời gian tạo |
| updatedAt | Date |  | Yes | Thời gian cập nhật |

---
## 5. TransField

**Purpose:** Khai báo các field cần validate trong TRANSBODY.

| Field | Type | Key | Required | Description |
|---|---|---|---|---|
| id | String/ObjectId | PK | Yes | Khóa chính |
| service | String/ObjectId | FK | Yes | Tham chiếu đến Service.id |
| fieldName | String |  | Yes | Tên field, ví dụ SERVICEID, AMOUNT |
| fieldFormat | String |  | Yes | Kiểu dữ liệu: string/number/phone |
| minLength | Number |  | No | Độ dài tối thiểu |
| maxLength | Number |  | No | Độ dài tối đa |
| regex | String |  | No | Regex validate |
| isRequired | Boolean |  | Yes | Field có bắt buộc không |
| needSecured | Boolean |  | Yes | Có cần che/mã hóa không |
| order | Number |  | Yes | Thứ tự validate |
| errorCode | String |  | Yes | Mã lỗi khi validate fail |
| status | String |  | Yes | active/inactive |

---
## 6. TransValidation

**Purpose:** Khai báo các luật kiểm tra nghiệp vụ.

| Field | Type | Key | Required | Description |
|---|---|---|---|---|
| id | String/ObjectId | PK | Yes | Khóa chính |
| service | String/ObjectId | FK | Yes | Tham chiếu đến Service.id |
| validateFunc | String |  | Yes | Tên hàm validate |
| validateFields | String |  | Yes | Các field truyền vào hàm, cách nhau bằng dấu : |
| order | Number |  | Yes | Thứ tự chạy validate |
| errorCode | String |  | Yes | Mã lỗi khi validate fail |
| status | String |  | Yes | active/inactive |

---
## 7. TransDefinition

**Purpose:** Khai báo cách tiền di chuyển giữa các ví.

| Field | Type | Key | Required | Description |
|---|---|---|---|---|
| id | String/ObjectId | PK | Yes | Khóa chính |
| code | String/ObjectId | FK | Yes | Tham chiếu đến Service.id |
| glSteps | Array |  | Yes | Danh sách các bước chuyển tiền |
| status | String |  | Yes | active/inactive |
| createdAt | Date |  | Yes | Thời gian tạo |
| updatedAt | Date |  | Yes | Thời gian cập nhật |

**Note:** `TransDefinition.code = String(service._id)`, không phải `service.code`.

---
## 8. TransactionTrail

**Purpose:** Lưu hồ sơ giao dịch qua 3 bước request → confirm → verify.

| Field | Type | Key | Required | Description |
|---|---|---|---|---|
| id | String/ObjectId | PK | Yes | Khóa chính, đồng thời là transRefId |
| service | String/ObjectId | FK | Yes | Tham chiếu đến Service.id |
| user | String/ObjectId | FK | Yes | User tạo giao dịch |
| inputMessage | Object |  | Yes | Dữ liệu request ban đầu |
| outputMessage | Object |  | Yes | Message sau khi BuildMessage |
| transBody | Object |  | Yes | TRANSBODY đã chuẩn hóa |
| status | String |  | Yes | init/pending/done/failed |
| errorCode | String |  | No | Mã lỗi nếu giao dịch thất bại |
| createdAt | Date |  | Yes | Thời gian tạo |
| updatedAt | Date |  | Yes | Thời gian cập nhật |

---
## 10. Transaction

**Purpose:** Lưu biên lai giao dịch cuối cùng sau khi verify.

| Field | Type | Key | Required | Description |
|---|---|---|---|---|
| id | String/ObjectId | PK | Yes | Khóa chính |
| code | String | Unique | Yes | Mã biên lai giao dịch |
| transRefId | String/ObjectId | FK | Yes | Tham chiếu đến TransactionTrail.id |
| service | String/ObjectId | FK | Yes | Tham chiếu đến Service.id |
| sender | String/ObjectId | FK | Yes | Ví/user gửi |
| receiver | String/ObjectId | FK | Yes | Ví/user nhận |
| amount | Number |  | Yes | Số tiền gốc |
| fee | Number |  | Yes | Phí giao dịch |
| totalAmount | Number |  | Yes | Tổng tiền bị trừ |
| status | String |  | Yes | done/failed |
| createdAt | Date |  | Yes | Thời gian tạo |

---
## 11. Biller

**Purpose:** Lưu nhà cung cấp dịch vụ thanh toán hóa đơn.

| Field | Type | Key | Required | Description |
|---|---|---|---|---|
| id | String/ObjectId | PK | Yes | Khóa chính |
| code | String | Unique | Yes | Mã biller, ví dụ EVN |
| name | String |  | Yes | Tên biller |
| inquiryUrl | String |  | Yes | API tra cứu hóa đơn |
| paymentUrl | String |  | Yes | API thanh toán hóa đơn |
| pocketId | String/ObjectId | FK | Yes | Ví nhận tiền của biller, tham chiếu Pocket.id |
| status | String |  | Yes | active/inactive |
| createdAt | Date |  | Yes | Thời gian tạo |
| updatedAt | Date |  | Yes | Thời gian cập nhật |

---

## 12. Bill

**Purpose:** Lưu hóa đơn mẫu dùng cho nghiệp vụ Bill Payment.

| Field | Type | Key | Required | Description |
|---|---|---|---|---|
| id | String/ObjectId | PK | Yes | Khóa chính |
| billerId | String/ObjectId | FK | Yes | Tham chiếu đến Biller.id |
| customerCode | String |  | Yes | Mã khách hàng hoặc mã hóa đơn |
| amount | Number |  | Yes | Số tiền hóa đơn |
| status | String |  | Yes | unpaid/paid/expired |
| paidTransRefId | String/ObjectId | FK | No | Giao dịch đã thanh toán, tham chiếu TransactionTrail.id |
| createdAt | Date |  | Yes | Thời gian tạo |
| updatedAt | Date |  | Yes | Thời gian cập nhật |

---
## 13. Important Notes

- `Pocket.balance` là số dư hiện tại của ví.
- `PocketEntry` là lịch sử bút toán, không phải nơi lưu số dư.
- `TransactionTrail.id` chính là `transRefId`.
- `requestTransaction` chỉ tạo `TransactionTrail` ở trạng thái `pending`.
- Tiền chỉ thay đổi ở `verifyTransaction`.
- `TransDefinition.ExecuteTransaction` sẽ tạo `PocketEntry` và `Transaction`.
- `TransField` của mỗi service bắt buộc có `SERVICEID`.
- Khóa ngoại config dùng `String(service._id)`, không dùng `service.code`.