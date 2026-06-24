# Mini-Wallet API Design

## 1. Authentication APIs
|Method|Endpoint|Purpose|
|---|---|---|
|POST|/api/auth/login|Đăng nhập user/officer/admin|
|POST|/api/auth/logout|Đăng xuất user/officer/admin|
|POST|/api/auth/set-pin|Tạo hoặc đổi mã Pin|
|GET|/api/auth/me|Lấy thông tin user hiện tại|
|POST|/api/auth/verify-pin|Kiểm tra PIN khi verify giao dịch|

## 2.Wallet APIs
|Method|Endpoint|Purpose|
|---|---|---|
|GET|/api/wallet/me|Xem ví của user hiện tại|
|GET|/api/transactions/me|Xem lịch sử ghi sổ của ví|

## 3.Transaction APIs
### 3.1.Request Transaction
|Method|Endpoint|Purpose|
|---|---|---|
| POST | /api/transactions/request | Khởi tạo giao dịch, validate input, tính phí, trả preview |

Input chính:
- serviceCode
- parameters

Output chính:
- transRefId
- amount
- fee
- totalAmount
- status

### 3.2.Confirm Transaction
|Method|Endpoint|Purpose|
|---|---|---|
| POST | /api/transactions/confirm |Xác nhận giao dịch và trả về phương thức xác thực|

Input chính:
- transRefId

Output chính:
- transRefId
- authMethod: PIN hoặc NONE

### 3.3.Verify Transaction
|Method|Endpoint|Purpose|
|---|---|---|
| POST | /api/transactions/verify |Xác thực và thực thi giao dịch|

Input chính:
- transRefId
- pin nếu authMethod = PIN

Output chính:
- transactionId
- transRefId
- amount
- fee
- totalAmount
- status

Note: Tiền chỉ được trừ/cộng ở bước verify, thông qua ExecuteTransaction trong session.withTransaction.V

## 4.Admin APIs
|Method|Endpoint|Purpose|
|---|---|---|
| GET | /api/admin/users |Xem danh sách user|
|GET|/api/admin/pockets|Xem danh sách ví|
|GET|/api/admin/transactions|Xem danh sách giao dịch|
|GET|/api/admin/services|Xem danh sách service|
|POST|/api/admin/services|Tao service|
|PATCH|/api/admin/services/:id|Cập nhật service|
|GET|/api/admin/trans-fields|Xem cấu hình transfields|
|POST|/api/admin/trans-fields|Tạo transfields|
|POST|/api/admin/trans-validations|Tạo transValidation|
|GET|/api/admin/trans-validations|Xem transValidation|
|POST|/api/admin/trans-definitions|Tạo transDefinition|
|GET|/api/admin/trans-definitions|Xem transDefinition|
|GET|/api/admin/billers|Xem danh sách biller|
|POST|/api/admin/billers|Tạo biller|
|POST|/api/admin/bills|Tạo hóa đơn mẫu|
|GET|/api/admin/bills|Xem danh sách hóa đơn mẫu|







