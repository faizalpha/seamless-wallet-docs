# Seamless Wallet Service Flow

Diagram berikut menjelaskan arsitektur dan alur transaksi pada layanan wallet seamless.

```mermaid
flowchart TD

%% --- Entry Layer ---
A[Game Provider (Pragmatic, etc)] -->|API / gRPC| B[Wallet Service (Multi Node)]
subgraph "Wallet Service"
    B1[1. Receive Transaction Request]
    B2[2. Idempotency Check (Redis SETNX idemp:&lt;tx_id&gt;)]
    B3[3. Insert trx_log (INSERT ... ON CONFLICT DO NOTHING)]
    B4[4. Update Balance Atomic (UPDATE wallet SET balance=balance+amt WHERE balance+amt&gt;=0 RETURNING balance)]
    B5[5. Invalidate/Update Cache (Redis: DEL wallet:balance:&lt;player_id&gt;)]
    B6[6. Return Result (success / duplicate / insufficient)]
end

B --> B1 --> B2 --> B3 --> B4 --> B5 --> B6
B6 -->|Response| A

%% --- Data Layer ---
subgraph "Database Layer"
    D1[(PostgreSQL)]
    D1a[wallet table (player_id PK, balance, version)]
    D1b[trx_log table (unique tx_id, player_id, amount, provider)]
end

subgraph "Cache Layer"
    R1[(Redis)]
    R1a[idemp:&lt;tx_id&gt; key]
    R1b[wallet:balance:&lt;player_id&gt; cache]
end

B3 -->|INSERT / ON CONFLICT| D1b
B4 -->|Atomic UPDATE ... RETURNING| D1a
B5 -->|SET / DEL / EXPIRE| R1b
B2 -->|SETNX / EX| R1a

%% --- Background Process ---
subgraph "Background Jobs"
    J1[Periodic Reconciliation (DB vs Cache)]
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
