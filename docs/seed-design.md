# Mini-Wallet Seed Design


# 1. Currency Seed

Dữ liệu tiền tệ mặc định của hệ thống.

| Field  | Value        |
| ------ | ------------ |
| code   | MMK          |
| name   | Myanmar Kyat |
| status | active       |

```json
{
  "code": "MMK",
  "name": "Myanmar Kyat",
  "status": "active"
}
```

---

# 2. System Pocket Seed

Ví hệ thống dùng để nhận phí giao dịch.

| Field    | Value            |
| -------- | ---------------- |
| id       | SYSTEM_POCKET_ID |
| client   | system           |
| currency | MMK              |
| balance  | 0                |
| status   | active           |

```json
{
  "id": "SYSTEM_POCKET_ID",
  "client": "system",
  "currency": "MMK",
  "balance": 0,
  "checksum": "<generated>",
  "status": "active"
}
```

---

# 3. Bank Pocket Seed

Ví ngân hàng dùng để thực hiện nghiệp vụ Cash-in.

| Field    | Value          |
| -------- | -------------- |
| id       | BANK_POCKET_ID |
| client   | bank           |
| currency | MMK            |
| balance  | 100000000      |
| status   | active         |

```json
{
  "id": "BANK_POCKET_ID",
  "client": "bank",
  "currency": "MMK",
  "balance": 100000000,
  "checksum": "<generated>",
  "status": "active"
}
```

---

# 4. Officer Seed

Officer là nhân viên thực hiện Cash-in.

| Field    | Value        |
| -------- | ------------ |
| id       | OFFICER_001  |
| fullName | Demo Officer |
| phone    | 09900000001  |
| role     | officer      |
| status   | active       |

```json
{
  "id": "OFFICER_001",
  "fullName": "Demo Officer",
  "phone": "09900000001",
  "role": "officer",
  "state": "normal",
  "status": "active"
}
```

---

# 5. Customer Seeds

## Customer A

```json
{
  "id": "USER_A",
  "fullName": "Nguyen Van A",
  "phone": "0911111111",
  "role": "customer",
  "state": "normal",
  "status": "active"
}
```

### Pocket A

```json
{
  "id": "POCKET_A",
  "user": "USER_A",
  "client": "customer",
  "currency": "MMK",
  "balance": 500000,
  "checksum": "<generated>",
  "status": "active"
}
```

---

---

# 6. Biller Seed

Ví dụ biller EVN.

| Field  | Value      |
| ------ | ---------- |
| id     | BILLER_EVN |
| code   | EVN        |
| status | active     |

```json
{
  "id": "BILLER_EVN",
  "code": "EVN",
  "name": "Electricity Provider",
  "inquiryUrl": "/mock/billers/evn/inquiry",
  "paymentUrl": "/mock/billers/evn/payment",
  "pocketId": "EVN_POCKET_ID",
  "status": "active"
}
```

---

# 7. Biller Pocket Seed

Ví nhận tiền của biller.

```json
{
  "id": "EVN_POCKET_ID",
  "client": "biller",
  "currency": "MMK",
  "balance": 0,
  "checksum": "<generated>",
  "status": "active"
}
```

---

# 8. Sample Bill Seed

Hóa đơn mẫu phục vụ kiểm thử Bill Payment.

## Bill 1

```json
{
  "id": "BILL_001",
  "billerId": "BILLER_EVN",
  "customerCode": "EVN-CUS-001",
  "amount": 80000,
  "status": "unpaid",
  "paidTransRefId": null
}
```

## Bill 2

```json
{
  "id": "BILL_002",
  "billerId": "BILLER_EVN",
  "customerCode": "EVN-CUS-002",
  "amount": 120000,
  "status": "unpaid",
  "paidTransRefId": null
}
```

---

# 9. Testing Scenarios

## P2P Transfer

Sender:

```text
USER_A
Balance = 500,000 MMK
```

Receiver:

```text
USER_B
Balance = 100,000 MMK
```

Transfer:

```text
Amount = 10,000 MMK
Fee = 100 MMK
```

Expected:

```text
POCKET_A = 489,900
POCKET_B = 110,000
SYSTEM_POCKET = +100
```

---

## Cash-in

Officer thực hiện Cash-in:

```text
BANK_POCKET -> POCKET_A
Amount = 50,000 MMK
```

Expected:

```text
BANK_POCKET giảm 50,000
POCKET_A tăng 50,000
```

---

## Bill Payment

Customer:

```text
USER_A
```

Bill:

```text
BILL_001
Amount = 80,000 MMK
Fee = 100 MMK
```

Expected:

```text
POCKET_A giảm 80,100
EVN_POCKET tăng 80,000
SYSTEM_POCKET tăng 100
Bill status = paid
```

---

