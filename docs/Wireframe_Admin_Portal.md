## 1.Screen Dashboard
### Mục đích: hiển thị tổng quan hệ thống
#### Components

+ Total Users
+ Total Wallets
+ Total Transactions
+ Total Billers
+ Recent Transactions Table

#### Actions

+ View Users

+ View Wallets

+ View Transactions

## 2.Screen User Management
### Mục đích: Quản lý người dùng trong hệ thống.

#### Components
+ Search User
+ User List Table

|User ID|Full Name|Phone Number|Status|Created At|Actions|
|---|---|---|---|---|---|

#### Actions
+ View User Detail
+ Lock User
+ Unlock User

## 3.Screen Pocket Management
### Mục đích: Quản lý Ví và số dư

#### Components
+ Pocket List Table

|Pocket ID|Owner|Client Type|Currency|Balance|Status|
|---|---|---|---|---|---|

#### Actions
+ View Pocket Detail
+ View Pocket Entries

## 4.Screen Transaction Management
### Mục đích: Tra cứu giao dịch.

#### Components
+ Search Transaction
+ Transaction List Table

|Transaction ID|Service|Sender|Receiver|Amount|Fee|Status|Created At|
|---|---|---|---|---|---|---|---|

#### Actions
+ View Transaction Detail


## 5.Screen Transaction Detail
### Mục đích: Xem chi tiết một giao dịch và các bút toán phát sinh.

#### Components

+ Transaction Information

 Transaction ID
 Service
 Sender
 Receiver
 Amount
 Fee
 Total Amount
 Status

+ Pocket Entries

 Step Order
 Debit Pocket
 Credit Pocket
 Amount

+ Transaction Trail

 Request Time
 Confirm Time
 Verify Time

## 6.Screen Service Configuration
### Mục đích: Quản lý cấu hình nghiệp vụ.

#### Components
- Service List
- TransField List
- TransValidation List
- TransDefinition List
#### Actions
- Create Service
- Update Service
- Create TransField
- Create Validation
- Create Definition
#### Services
- P2P
- Cash-In
- Bill Payment

## 7.Screen Biller Management
### Mục đích: Quản lý biller và hóa đơn mẫu.

#### Components

- Biller List

|Biller ID|Name|Inquiry URL|Payment URL|Status|
|---|---|---|---|---|

- Bill List

|Bill ID|Customer Code|Amount|Status|
|---|---|---|---|

#### Actions
- Create Biller
- Update Biller
- Create Sample Bill
- View Bill Detail