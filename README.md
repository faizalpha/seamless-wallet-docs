flowchart TD

    %% --- Entry Layer ---
    A[Game Provider (Pragmatic, etc)] -->|API / gRPC| B[Wallet Service (Multi Node)]
    subgraph "Wallet Service"
        B1[1️⃣ Receive Transaction Request]
        B2[2️⃣ Idempotency Check\nRedis SETNX idemp:<tx_id>]
        B3[3️⃣ Insert trx_log\nINSERT ... ON CONFLICT DO NOTHING]
        B4[4️⃣ Update Balance Atomic\nUPDATE wallet SET balance=balance+amt\nWHERE balance+amt>=0 RETURNING balance]
        B5[5️⃣ Invalidate Cache / Update Cache\nRedis: DEL wallet:balance:<player_id>]
        B6[6️⃣ Return Result\n(success / duplicate / insufficient)]
    end

    B --> B1 --> B2 --> B3 --> B4 --> B5 --> B6
    B6 -->|Response| A

    %% --- Data Layer ---
    subgraph "Database Layer"
        D1[(PostgreSQL)]
        D1a[wallet table\nplayer_id PK, balance, version]
        D1b[trx_log table\nunique tx_id, player_id, amount, provider]
    end

    subgraph "Cache Layer"
        R1[(Redis)]
        R1a[idemp:<tx_id> key]
        R1b[wallet:balance:<player_id> cache]
    end

    B3 -->|INSERT / ON CONFLICT| D1b
    B4 -->|Atomic UPDATE ... RETURNING| D1a
    B5 -->|SET / DEL / EXPIRE| R1b
    B2 -->|SETNX / EX| R1a

    %% --- Background Process ---
    subgraph "Background Jobs"
        J1[Periodic Reconciliation\n(DB vs Cache)]
        J2[Reapply Failed Transactions]
        J3[Cleanup Expired Idempotency Keys]
    end
    J1 --> D1a
    J1 --> R1b
    J2 --> D1b
    J3 --> R1a

    %% --- Monitoring ---
    subgraph "Observability & Metrics"
        M1[Log Duplicates]
        M2[Track Update Latency]
        M3[Reconciliation Drift Alert]
    end
    D1b --> M1
    D1a --> M2
    J1 --> M3

    style A fill:#f6d365,stroke:#e6b200,stroke-width:2px
    style B fill:#c2e9fb,stroke:#0088cc,stroke-width:2px
    style D1 fill:#d4e157,stroke:#8bc34a,stroke-width:2px
    style R1 fill:#ffd54f,stroke:#ff9800,stroke-width:2px
    style J1 fill:#b39ddb,stroke:#7e57c2,stroke-width:2px
    style M1 fill:#f8bbd0,stroke:#e91e63,stroke-width:2px
