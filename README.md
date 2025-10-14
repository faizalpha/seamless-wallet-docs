# Seamless Wallet Service Flow (Simplified)

Diagram berikut menjelaskan arsitektur dan alur transaksi utama pada layanan wallet seamless — tanpa cache layer dan tanpa background process.

```mermaid
flowchart TD

%% --- Entry Layer ---
A[Game Provider - Pragmatic etc] -->|gRPC API| B[Wallet Service Multi Node]

subgraph WalletService
    B1[1. Receive Transaction Request]
    B2[2. Idempotency Check - Redis SETNX idemp:&lt;tx_id&gt;]
    B3[3. Insert trx_log - INSERT ON CONFLICT DO NOTHING]
    B4[4. Update Balance Atomic - UPDATE wallet SET balance=balance+amt WHERE balance+amt&gt;=0 RETURNING balance]
    B5[5. Return Result - success duplicate insufficient]
end

A --> B1 --> B2 --> B3 --> B4 --> B5
B5 -->|Response| A

%% --- Data Layer ---
subgraph DatabaseLayer
    D1[(PostgreSQL)]
    D1a[wallet table - player_id PK, balance, version]
    D1b[trx_log table - unique tx_id, player_id, amount, provider]
end

%% --- Redis for Idempotency Only ---
R1[(Redis)]
R1a[idemp:&lt;tx_id&gt; key]

B3 --> D1b
B4 --> D1a
B2 --> R1a
