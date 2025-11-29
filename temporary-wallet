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

sequenceDiagram
    participant U as User
    participant API as Wallet API
    participant TW as Temporary Wallet
    participant CW as Common Wallet

    U->>API: Request claim amount
    API->>TW: Validasi balance temporary
    TW-->>API: Balance OK / Not OK

    alt Balance Tidak Cukup
        API-->>U: Error (insufficient temporary balance)
    else Balance Cukup
        API->>TW: Deduct temporary balance
        TW-->>API: OK
        API->>CW: Add to common wallet
        CW-->>API: OK
        API-->>U: Claim success
    end

erDiagram
    MEMBER ||--o{ WALLET_COMMON : owns
    MEMBER ||--o{ WALLET_PROMO : owns
    MEMBER ||--o{ WALLET_TEMPORARY : owns

    WALLET_COMMON {
        decimal balance
        datetime updated_at
    }

    WALLET_PROMO {
        decimal balance
        datetime updated_at
    }

    WALLET_TEMPORARY {
        decimal balance
        datetime updated_at
    }

    TRANSACTION ||--o{ WALLET_COMMON : affects
    TRANSACTION ||--o{ WALLET_PROMO : affects
    TRANSACTION ||--o{ WALLET_TEMPORARY : affects
