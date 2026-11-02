---
kata: AnkiX.DeadlockFreeLedger
status: locked           # draft | locked | complete
ankix-target: src/backend/AnkiX.Api/Services/AiCreditService.cs#L11-L545, src/backend/AnkiX.Api.Tests/AiCreditServiceTests.cs#L201-L243
study-group: csharp-concurrency-and-db-locks (not created yet)
---

# KATA-02: AnkiX.DeadlockFreeLedger — From C# Threads to Deadlock-Free Postgres Transfers

## Why this kata
`AiCreditService.cs` protects user AI credit balances against concurrent double-spends and peer-to-peer transfer deadlocks, but its existing unit tests in `AiCreditServiceTests.cs` only exercise an in-memory `SemaphoreSlim` fallback rather than the production PostgreSQL lock SQL. This kata rebuilds the three concurrency layers in a standalone C# + Docker Postgres workbook so every lock, race condition, and double-entry invariant is proven by failing-then-passing tests.

## Study Group scope
1. **In-Process C# Async Synchronization vs. `DbContext` Lifetime:** `Task.WhenAll` (`asyncio.gather`), `SemaphoreSlim(1, 1)` (`asyncio.Lock`), why EF Core `DbContext` is not thread-safe, and why in-process locks fail across multiple web servers.
2. **Single-Row Lost Updates vs. Atomic Conditional `UPDATE`:** Why read-then-write in application memory overdrafts under parallel requests, and how `UPDATE ... SET CreditBalance = CreditBalance - @cost WHERE UserId = @id AND CreditBalance >= @cost` serializes at the row tuple in PostgreSQL.
3. **Two-Row Circular Wait Deadlocks (`40P01`) vs. Deterministic Lock Ordering:** How bidirectional P2P transfers (`Alice -> Bob` and `Bob -> Alice`) deadlock when locking sender-first, and how sorting IDs ascending (`Math.Min(u1, u2)` + `SELECT ... ORDER BY UserId ASC FOR UPDATE`) breaks the cycle.
4. **The Phantom/Missing Row Lock Trap:** Why `SELECT ... FOR UPDATE` on a non-existent receiver row locks zero tuples, how `INSERT ... ON CONFLICT ("UserId") DO NOTHING` guarantees the tuple exists first, and what happens to lock order when the `INSERT` acquires a row lock before `FOR UPDATE`.
5. **Double-Entry Zero-Sum Conservation & Testing Fidelity:** Enforcing $\Delta \text{Sender} + \Delta \text{Receiver} = 0$ with immutable audit rows, and why `UseInMemoryDatabase` masks relational concurrency bugs.

## Layers

### Layer 1: In-Process Threads, `SemaphoreSlim`, and the Shared-`DbContext` Trap
- Anchor: `src/backend/AnkiX.Api/Services/AiCreditService.cs#L11`, `src/backend/AnkiX.Api/Services/AiCreditService.cs#L140-L176`, `src/backend/AnkiX.Api.Tests/AiCreditServiceTests.cs#L201-L243`
- Lab command: `dotnet test --filter "FullyQualifiedName~Layer1"`
- [ ] CLI challenge
- [ ] Study Group deck & drill (exercise ids: )
- [ ] Azimuth checkpoint
- [ ] Field report (draft: docs/publications/layer-1.md, URL: )

### Layer 2: Single-Row Database Races & Atomic Conditional `UPDATE`
- Anchor: `src/backend/AnkiX.Api/Services/AiCreditService.cs#L77-L138` (`DeductCreditsAsync`)
- Lab command: `dotnet test --filter "FullyQualifiedName~Layer2"`
- [ ] CLI challenge
- [ ] Study Group deck & drill (exercise ids: )
- [ ] Azimuth checkpoint
- [ ] Field report (draft: docs/publications/layer-2.md, URL: )

### Layer 3: Two-Row P2P Transfers, Missing-Row Upserts & Deadlock-Free `SELECT ... FOR UPDATE`
- Anchor: `src/backend/AnkiX.Api/Services/AiCreditService.cs#L422-L545` (`GetDeterministicLockOrder`, `TransferCreditsAsync`)
- Lab command: `dotnet test --filter "FullyQualifiedName~Layer3"`
- [ ] CLI challenge
- [ ] Study Group deck & drill (exercise ids: )
- [ ] Azimuth checkpoint
- [ ] Field report (draft: docs/publications/layer-3.md, URL: )

## Parked Azimuth Questions
<!-- Raised in expedition mode; answered in home mode against live ankiX. -->
- [ ] In `AiCreditService.cs#L475-L485`, when `receiver.Id < senderUserId` and the receiver has no `UserAiBalances` row yet, does `INSERT ... ON CONFLICT DO NOTHING` acquire a row lock only on the lower ID first (safe), or what if `receiver.Id > senderUserId` while another transaction concurrently locks `senderUserId` first? Verify whether both sender and receiver should be upserted before `SELECT ... FOR UPDATE`. (layer 3, raised 2026-11-01)

## Pay It Back to AnkiX
- [ ] Add `?id=<exerciseId>` deep-link support in `src/frontend/src/pages/Exercises.jsx` so Tales from the Field articles can open a specific exercise directly.
- [ ] Add relational concurrency tests for `DeductCreditsAsync` and `TransferCreditsAsync` in `src/backend/AnkiX.Api.Tests/AiCreditServiceTests.cs` using isolated `DbContext` instances per parallel task rather than a single shared `UseInMemoryDatabase` context.
- [ ] Apply any lock-order refinement to `TransferCreditsAsync` in `src/backend/AnkiX.Api/Services/AiCreditService.cs#L475-L485` if Layer 3 proves a race on first-time receiver row creation.

## Decisions
- 2026-11-01: Scoped to 3 layers in C# + Docker Postgres (`AnkiX.DeadlockFreeLedger`), backed by Python Rosetta Stone comparisons and an AnkiX Study Group funnel.

## Sync log
- 2026-11-01: home-pc, plan drafted (uncommitted)
