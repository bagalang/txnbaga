# txnbaga — gaps

## T1 — N = 2, same write set on all participants

Demo replication, not key-partitioned shards. Coordinator would route
PREPARE by key range in a real system.

## T2 — no durable coordinator log

Crash of the coordinator between PREPARE and COMMIT is unrecoverable
(classic 2PC). Presumed-abort + WAL is future work (pairs with lsmbaga).

## T3 — MVCC GC missing — **closed 2026-08-24**

**Shipped.** `mvcc_gc(store, keep_ts)` drops versions older than the
newest one with `ts <= keep_ts`. Newer versions stay. `keep_ts` is the
oldest live snapshot.

## T4 — max 2 keys per txn

Message packing limit; general write-sets need a list encoding.

## T5 — concurrent PREPARE (improved B4.3)

**Shipped.** Multiple concurrent PREPARE allowed when write-sets do not
lock-conflict (`Map` of preps by tx). Conflicting keys still VOTE_NO (no
wait queue). Stress: `tests/txn_stress_test.baga` (go_bg clients + hot key).

**Residual.** No wait queue / deadlock detection; client retries are
app-level (`relbaga`).
