## 1. P2P config
Khách nhập SĐT người nhận + số tiền, phí cố định 100.

fieldBuilder:
CURRENCY = fixed "MMK" ·
RECEIVERPHONE = mapping parameters.receiverPhone ·
AMOUNT = mapping parameters.amount ·
SENDERID = query queryPocketByUserId(USERID).id ·
RECEIVERID = query queryPocketByPhone(RECEIVERPHONE).id

TransField:
SERVICEID bắt buộc ·
RECEIVERPHONE string, regex số ·
AMOUNT number

TransValidation:
validateReceiverIsNotSender(SENDERID:RECEIVERID) ·
validateSenderAccountSufficiency(SENDERID:AMOUNT:DEBITFEE)

Phí/auth:
fee: { type: 'fixed', value: 100 } ·
auth: { method: 'PIN' }

glSteps:

|order|amount|debit|credit|
|---|---|---|---|
|0|AMOUNT||ProductLevel SENDERID|ProductLevel RECEIVERID|
|0|FEE||ProductLevel SENDERID|WALLET SYSTEM POCKET|

## 2. Cash-in config
Officer nhập SĐT người nhận + số tiền

fieldBuilder:
CURRENCY = fixed "MMK" ·
RECEIVERPHONE = mapping parameters.receiverPhone ·
AMOUNT = mapping parameters.amount
SENDERID =fixed BANK_POCKET_ID
RECEIVERID = query queryPocketByPhone(RECEIVERPHONE).id

TransField:
SERVICEID bắt buộc ·
RECEIVERPHONE string, regex số ·
AMOUNT number

TransValidation:
validateReceiverPocketActive(RECEIVERID) ·
validateSenderAccountSufficiency(SENDERID:AMOUNT:DEBITFEE)

Phí/auth:
fee: { type: 'fixed', value: 0 } ·
auth: { method: 'NONE' }

glSteps: 
|order|amount|debit|credit|
|---|---|---|---|
|0|AMOUNT||BANK_POCKET_ID|ProductLevel RECEIVERID|

## 3. Bill Payment config
Khách nhập billerId + mã hóa đơn/customerCode, không nhập số tiền. Amount lấy từ inquiryUrl ở bước Request. Phí cố định 100.

fieldBuilder:
CURRENCY = fixed "MMK" ·
BILLERID: mapping parameters.billerId
CUSTOMERCODE:  mapping parameters.customerCode
AMOUNT = query inquiryBillAmount(BILLERID:CUSTOMERCODE)
SENDERID = query queryPocketByUserId(USERID).id
RECEIVERID = query queryBillerByBillerID(BILLERID).id

TransField:
SERVICEID bắt buộc ·
BILLERID string ·
CUSTOMERCODE string ·
AMOUNT number

TransValidation:
validateBillerActive(BILLERID) ·
validateBillUnpaid(BILLERID:CUSTOMERCODE) ·
validateSenderAccountSufficiency(SENDERID:AMOUNT:DEBITFEE)

Phí/auth:
fee: { type: 'fixed', value: 100 } ·
auth: { method: 'PIN' } ·
action: 'billerTrans'

glSteps:
|order|amount|debit|credit|
|---|---|---|---|
|0|AMOUNT|ProductLevel SENDERID|productLevel RECEIVERID|
|1|DEBITFEE|ProductLevel SENDERID|WALLET SYSTEM POCKET|



Request: gọi inquiryUrl để lấy AMOUNT.

Verify: gọi paymentUrl trước khi ghi sổ.

Nếu biller fail → Trail failed, không tạo PocketEntry.

Nếu biller success → ExecuteTransaction chạy tiền.

Gửi transRefId sang biller để idempotent.