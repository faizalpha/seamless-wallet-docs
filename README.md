# Seamless Wallet Service Flow

Diagram berikut menjelaskan arsitektur dan alur transaksi pada layanan wallet seamless.

```mermaid
flowchart TD

%% --- Entry Layer ---
A[Game Provider - Pragmatic etc] -->|gRPC API| B[Wallet Service Multi Node]

subgraph WalletService
    B1[1. Receive Transaction Request]
    B2[2. Idempotency Check - Redis SETNX idemp:&lt;tx_id&gt;]
    B3[3. Insert trx_log - INSERT ON CONFLICT DO NOTHING]
    B4[4. Update Balance Atomic - UPDATE wallet SET balance=balance+amt WHERE balance+amt&gt;=0 RETURNING balance]
    B5[5. Invalidate or Update Cache - Redis DEL wallet:balance:&lt;player_id&gt;]
    B6[6. Return Result - success duplicate insufficient]
end

A --> B1 --> B2 --> B3 --> B4 --> B5 --> B6
B6 -->|Response| A

%% --- Data Layer ---
subgraph DatabaseLayer
    D1[(PostgreSQL)]
    D1a[wallet table - player_id PK, balance, version]
    D1b[trx_log table - unique tx_id, player_id, amount, provider]
end

subgraph CacheLayer
    R1[(Redis)]
    R1a[idemp:&lt;tx_id&gt; key]
    R1b[wallet:balance:&lt;player_id&gt; cache]
end

B3 --> D1b
B4 --> D1a
B5 --> R1b
B2 --> R1a

%% --- Background Process ---
subgraph BackgroundJobs
    J1[Periodic Reconciliation DB vs Cache]
    J2[Reapply Failed Transactions]
    J3[Cleanup Expired Idempotency Keys]
end
J1 --> D1a
J1 --> R1b
J2 --> D1b
J3 --> R1a

%% --- Monitoring ---
subgraph Observability
    M1[Log Duplicates]
    M2[Track Update Latency]
    M3[Reconciliation Drift Alert]
end
D1b --> M1
D1a --> M2
J1 --> M3
