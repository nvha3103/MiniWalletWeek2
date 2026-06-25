# Generic Transaction Engine Design
# 1.Các config chính của engine
## 1.1. Service

Service mô tả một loại nghiệp vụ.Ví dụ: P2P Transfer, Cash-in, Bill Payment, Bank Transfer

Service chứa:

- code
- name
- fieldBuilder
- fee
- auth
- action
- status

Ví dụ:
```json
{
  code: "P2P",
  name: "P2P Transfer",
  fee: { type: "fixed", value: 100 },
  auth: { method: "PIN" },
  action: "none",
  status: "active"
}
```

## 1.2. fieldBuider
fieldBuilder là config dùng để build dữ liệu giao dịch thật từ request đầu vào.

Ví dụ: P2P user chỉ nhập các thông tin như: receiverPhone, amount. Nhưng engine lại cần thêm: CURRENCY
SENDERID, RECEIVERID. 

Vì vậy fieldBuilder sẽ định nghĩa cách tạo ra các field đó:
- CURRENCY = fixed "MMK"
- RECEIVERPHONE = mapping parameters.receiverPhone 
- AMOUNT = mapping parameters.amount 
- SENDERID = query queryPocketByUserId(USERID).id 
- RECEIVERID = query queryPocketByPhone(RECEIVERPHONE).id

Sau khi build xong, engine sẽ tạo ra TRANSBODY



## 1.3. TransField
TransField sẽ dùng để validate format dữ liệu trong TRANSBODY

Ví dụ:
- SERVICEID: required string 
- RECEIVERPHONE: required string, regex phone 
- AMOUNT: required number

Nó trả lời câu hỏi: Dữ liệu có đúng kiểu, đúng định dạng, đúng độ dài không?

## 1.4. TransValidate
Dùng để validate luật nghiệp vụ

Ví dụ với P2P ta có:
- validateReceiverIsNotSender(SENDERID:RECEIVERID)
- validateSenderAccountSufficiency(SENDERID:AMOUNT:DEBITFEE)

Nó trả lời câu hỏi: Giao dịch này có được phép thực hiện không?

## 1.5. TransDefinition
Dùng để mô tả luồng tiền 

Ví dụ P2P:

|order	|amount|	debit|	credit|
|---|---|---|---|
|0	|AMOUNT	|SENDERID	|RECEIVERID|
|1	|DEBITFEE	|SENDERID	|SYSTEM_POCKET_ID|

Nó trả lời câu hỏi: Nếu giao dịch được phép, tiền sẽ chạy từ ví nào sang ví nào?

## 1.6. External Action
Để engine trở nên tổng quát hơn, có thể bổ sung ExternalAction. ExternalAction mô tả việc gọi hệ thống bên ngoài như Bill inquiry, Bill Payment..

Cấu hình mẫu:
```json
{ 
    service: "BILL_PAYMENT", 
    phase: "REQUEST" | "VERIFY_AFTER_LEDGER" | "VERIFY_BEFORE_LEDGER", 
    type: "HTTP", 
    endpoint: "/mock/billers/evn/payment", 
    timeoutMs: 10000, 
    retryPolicy: { maxRetries: 3, delayMs: 5000 }, 
    idempotencyKey: "transRefId", 
    onSuccessStatus: "done", 
    onFailStatus: "pending_reversal" 
}
```

# 2. Ba runtime chính của engine
## 2.1. Runtime requestTransaction
Mục tiêu của request là tạo preview giao dịch, chưa thực hiện trừ tiền 

Luồng hoạt động: 
1. Nhận serviceCode, parameters, clientRequestId
2. Kiểm tra duplicate request bằng clientRequestId
3. Load service config từ database
4. Build TRANSBODY từ fieldBuider
5. Tạo TransactionTrail status = init
6. Validate TransField
7. Chạy preAction nếu có, ví dụ:Biller inquiry
8. Tính phí từ Service.fee
9. Validate TransValidation
10. Update TransactionTrail status=pending
11. Trả preview gồm transRefId, amount, fee, totalAmount

## 2.2. Runtime confirmTransaction
Mục tiêu: cho frontend biết cần xác thực kiểu gì

Luồng hoạt động:
1. Nhận transRefId
2. Load TransactionTrail có status=pending
3. Đọc Service config
4. Đọc service.auth
5. Trả authMethod

## 2.3. Runtime verifyTransaction
Mục tiêu: xác thực và thực thi giao dịch

Luồng hoạt động:
1. Nhận transRefId và Pin nếu cần
2. Load Transaction trail
3. Kiểm tra idempotency bằng transRefId
4. Lock sender account
5. Verify PIN nếu auth=PIN
6. Validate lại TransField
7. Tính lại phí 
8. Validate lại TransValidation
9. ExecuteTransaction theo TransDefinition.glSteps
10. Gọi ExternalAction nếu có
11. Update trạng thái cuối cùng của TransactionTrail 
12. Release sender account

Chú ý: Verify không được tin hoàn toàn dữ liệu từ Request. Vì giữa Request và Verify, số dư, phí hoặc trạng thái tài khoản có thể đã thay đổi.

## 2.4. ExecuteTransaction
ExecuteTransaction là nơi tiền thật sự chạy.

Toàn bộ bước này phải nằm trong session.withTransaction.

Luồng hoạt động:
1. Start database session.withTransaction
2. Loop qua từng glStep trong TransDefinition
3. Resolve amount từ AMOUNT, DEBITFEE hoặc biến tương ứng
4. Resolve debit pocket
5. Resolve credit pocket
6. Kiểm tra debit pocket đủ balance
7. Debit pocket bằng $inc
8. Credit pcoket bằng $inc
9. Recalculate checksum cho cả debit và credit pocket
10. Create pocketEntry cho glStep hiện tại
11. Sau khi chạy hết glSteps, create Transaction receipt
12. Update TransactionTrail sang trạng thái phù hợp
13. Commit transaction nếu tất cae thành công
14. RollBack nếu có bất kì lỗi nào


# 3. Xử lý duplicate request và idempotency
## 3.1. Duplicate request từ client
Frontend nên gửi thêm clientRequestId.

Engine sẽ tạo unique key: userId + serviceCode + clientRequestId

Nếu user spam nút hoặc request bị timeout rồi gửi lại, engine kiểm tra:
Nếu đã có TransactionTrail tương ứng -> không tạo trail mới -> trả lại trail cũ


## 3.2. Duplicate verify
Nếu frontend gọi verifyTransaction nhiều lần với cùng transRefId:

Trail pending → xử lý
Trail verifying → trả processing
Trail ledger_done/external_processing → trả processing
Trail done → trả receipt cũ
Trail failed → trả lỗi cũ

Nhờ vậy, cùng một giao dịch không bị ExecuteTransaction nhiều lần.

## 3.3.Idempotency khi gọi external provider
Khi gọi biller hoặc bank, Mini-Wallet gửi transRefId làm idempotency key.

Ví dụ:

paymentUrl(transRefId = TR00001)

Nếu Mini-Wallet gọi lại do timeout, provider phải hiểu đây là cùng một giao dịch và không xử lý trùng.

# 4. Xử lý timeout khi gọi biller hoặc bank
Với các nghiệp vụ có hệ thống ngoài như Bill Payment hoặc Bank Transfer:

Nếu external API trả success
→ update Trail done

Nếu external API trả failed
→ update Trail pending_reversal hoặc failed tùy tiền đã chạy chưa

Nếu external API timeout
→ không đánh failed ngay
→ update Trail external_unknown
→ tạo retry job hoặc status-check job

Lý do:

Timeout không có nghĩa là thất bại. Có thể provider đã xử lý thành công nhưng response bị mất.
