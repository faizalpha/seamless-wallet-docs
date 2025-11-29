```mermaid
flowchart TD
    A[User melakukan aktivitas] --> B{Sumber Balance?}

    B -->|Deposit Common| C[Masuk ke Common Wallet]
    B -->|Deposit Promo / Claim Promo| D[Masuk ke Promo Wallet]
    B -->|Cashback| E[Masuk ke Temporary Wallet]
    B -->|Referral| E
    B -->|Rebate| E

    %% Temporary wallet flow
    E --> F[User membuka menu Claim Temporary Wallet]
    F --> G[User input amount yang ingin di-claim]
    G --> H{Amount Valid?}

    H -->|Tidak Valid| I[Reject & tampilkan error]
    H -->|Valid| J[Kurangi balance di Temporary Wallet]
    J --> K[Tambahkan balance ke Common Wallet]

    K --> L[Success Response]
