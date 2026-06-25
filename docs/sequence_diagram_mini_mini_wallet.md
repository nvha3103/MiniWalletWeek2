# Transfer Sequence Diagram


## Muc Tieu Luong

Nguoi dung dang nhap, nhap so dien thoai nguoi nhan va so tien, sau do he thong tao mot giao dich `pending`. Nguoi dung sang man hinh chi tiet, nhap `personalCode`, roi backend moi thuc hien tru tien vi gui, cong tien vi nhan va cap nhat transaction thanh `success`.

## API Lien Quan

| API | Controller | Vai tro |
|---|---|---|
| `POST /api/wallet/overview` | `WalletController.overview` | Lay vi hien tai cua nguoi dang nhap |
| `POST /api/wallet/transfer/request` | `WalletController.request` | Tao transaction `pending` |
| `POST /api/wallet/transfer/detail` | `WalletController.detail` | Lay chi tiet transaction de hien thi man confirm |
| `POST /api/wallet/transfer/confirm` | `WalletController.confirm` | Xac thuc personal code va chuyen tien that |

## Sequence Diagram Tong Quan

```mermaid
sequenceDiagram
    actor User as User
    participant Transfer as Transfer.jsx
    participant Detail as TransferDetail.jsx
    participant API as Sails Routes
    participant Wallet as WalletController
    participant Session as req.session
    participant Account as Account Model
    participant Pocket as Pocket Model
    participant Tx as Transaction Model
    participant Mongo as MongoDB Transaction

    User->>Transfer: Mo man hinh Transfer
    Transfer->>API: POST /api/wallet/overview<br/>credentials: include
    API->>Wallet: overview(req, res)
    Wallet->>Session: Doc accountId

    alt Chua dang nhap
        Wallet-->>Transfer: { err: 401, message: "please login" }
    else Da dang nhap
        Wallet->>Pocket: findOne({ account: accountId })
        Pocket-->>Wallet: pocket
        Wallet-->>Transfer: { err: 200, pocket }
    end

    User->>Transfer: Nhap receiver phone + amount
    User->>Transfer: Bam Continue
    Transfer->>Transfer: Validate frontend<br/>phone required, amount > 0, amount <= balance
    Transfer->>API: POST /api/wallet/transfer/request<br/>{ phoneNumber, amount }<br/>credentials: include
    API->>Wallet: request(req, res)
    Wallet->>Session: Doc accountId

    alt Chua dang nhap
        Wallet-->>Transfer: { err: 401, message: "please login" }
    else Receiver khong ton tai
        Wallet->>Account: findOne({ phoneNumber })
        Account-->>Wallet: null
        Wallet-->>Transfer: { err: 404, message: "Receiver phone number doesn't exist" }
    else Du lieu hop le
        Wallet->>Account: findOne({ phoneNumber })
        Account-->>Wallet: accountReceiver
        Wallet->>Pocket: findOne({ account: accountId })
        Pocket-->>Wallet: pocketSend
        Wallet->>Pocket: findOne({ account: accountReceiver.id })
        Pocket-->>Wallet: pocketReceiver
        Wallet->>Tx: create({ status: "pending", sender, receiver, amount, expiresAt })
        Tx-->>Wallet: transaction
        Wallet-->>Transfer: { err: 200, transaction, pocketSend }
        Transfer->>Detail: navigate("/transfer/confirm/:transactionId")
    end

    Detail->>API: POST /api/wallet/transfer/detail<br/>{ transactionId }<br/>credentials: include
    API->>Wallet: detail(req, res)
    Wallet->>Session: Doc accountId
    Wallet->>Tx: findOne({ id: transactionId })
    Tx-->>Wallet: transaction
    Wallet->>Account: findOne({ id: accountId })
    Account-->>Wallet: accountSender
    Wallet->>Account: findOne({ id: transaction.receiver })
    Account-->>Wallet: accountReceiver

    alt Transaction khong thuoc sender hien tai
        Wallet-->>Detail: { err: 401 }
    else Hop le
        Wallet-->>Detail: { err: 200, transaction: { ..., senderName, receiverName } }
        Detail->>Detail: Hien thi senderName, receiverName, amount, status, expiresAt
        Detail->>Detail: Chay countdown theo expiresAt
    end

    User->>Detail: Nhap personalCode
    User->>Detail: Bam Confirm transfer
    Detail->>API: POST /api/wallet/transfer/confirm<br/>{ transactionId, personalCode }<br/>credentials: include
    API->>Wallet: confirm(req, res)
    Wallet->>Session: Doc accountId
    Wallet->>Tx: findOne({ id: transactionId })
    Tx-->>Wallet: transaction
    Wallet->>Account: findOne({ id: accountId })
    Account-->>Wallet: account
    Wallet->>Wallet: bcrypt.compare(personalCode, account.personalCode)

    alt Personal code sai
        Wallet->>Tx: updateOne({ id: transactionId }).set({ failedAttempts + 1 })
        Wallet-->>Detail: { err: 401, message: "Personal code is incorrect" }
    else Transaction het han
        Wallet->>Tx: updateOne({ id: transactionId, status: "pending" }).set({ status: "expired" })
        Wallet-->>Detail: { err: 400, message: "Transaction has expired..." }
    else Personal code dung
        Wallet->>Mongo: startSession()
        Wallet->>Mongo: session.withTransaction()
        Mongo->>Tx: find pending transaction<br/>{ _id, sender, status: "pending", expiresAt > now }
        Mongo->>Pocket: debit sender pocket<br/>{ $inc: { balance: -amount } }<br/>where balance >= amount
        Mongo->>Pocket: credit receiver pocket<br/>{ $inc: { balance: amount } }
        Mongo->>Tx: update transaction status = "success"
        Mongo-->>Wallet: commit transaction
        Wallet->>Tx: findOne({ id: transactionId })
        Tx-->>Wallet: confirmedTransaction
        Wallet-->>Detail: { err: 200, message: "Transfer successfully", transaction }
        Detail->>Detail: Hien popup "Transfer successful"
        User->>Detail: Bam Back to wallet
        Detail->>Transfer: navigate("/wallet")
    end
```

