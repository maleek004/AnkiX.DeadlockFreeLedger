# AnkiX.DeadlockFreeLedger (KATA-02)

Hands-on C# + PostgreSQL concurrency workbook mined from the **AnkiX AI Credit Ledger** (`AiCreditService.cs`).

## 3-Layer Prerequisite Progression
1. **Layer 1 — In-Process Threads, `SemaphoreSlim`, and the Shared-`DbContext` Trap:** Why sharing an EF Core `DbContext` across `Task.Run` threads fails, how `SemaphoreSlim(1,1)` masks it on `UseInMemoryDatabase`, and why in-process locks fail across multiple web servers.
2. **Layer 2 — Single-Row Database Races & Atomic Conditional `UPDATE`:** Preventing double-spend overdrafts under 20 concurrent database connections using `UPDATE ... WHERE CreditBalance >= @cost`.
3. **Layer 3 — Two-Row P2P Transfers, Missing-Row Upserts & Deadlock-Free `SELECT ... FOR UPDATE`:** Reproducing PostgreSQL `40P01 (deadlock_detected)` on concurrent bidirectional transfers and eliminating it with `INSERT ... ON CONFLICT DO NOTHING` + ascending lock order (`Math.Min(u1, u2)`).

## Quick Start
```powershell
docker compose up -d
dotnet test
```

See [`plan.md`](./plan.md) for current layer progress and [`docs/ANKIX_VERTICAL_SLICE.md`](./docs/ANKIX_VERTICAL_SLICE.md) for how this kata maps to the AnkiX monolith.
