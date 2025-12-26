# FastAPI scenarios

### 1. Handle 10k concurrent requests efficiently
- **Approach:** Use an async stack end-to-end (Uvicorn/Hypercorn + FastAPI + async drivers), scale horizontally with multiple workers/pods, and keep handlers non-blocking.
- **Key optimizations:** Enable HTTP keep-alive, gzip, and use connection pooling; avoid per-request object creation; pre-warm caches; use uvloop where available.
- **Backpressure & resilience:** Tune timeouts, max headers/body size, and implement circuit breakers and retries; shed load via rate limiting and queueing for spillover.
- **Observability:** Measure P99 latency, saturation (CPU, event-loop lag), and connection pool metrics; load test with realistic patterns.
- **Follow-up question:** What metrics or signals would you prioritize to decide when to add more replicas vs optimize code paths?

### 2. Optimize slow endpoint with async/await and pooling
- **Diagnosis:** Confirm the slowness is I/O-bound via profiling and database EXPLAIN; identify N+1 queries and blocking calls.
- **Implementation:** Use async drivers (e.g., asyncpg/SQLAlchemy async) with a tuned pool; batch queries; leverage prepared statements and server-side cursors for large result sets.
- **Await discipline:** Ensure no synchronous CPU-heavy work in handlers; offload to background workers or threadpools if needed.
- **Caching:** Memoize hot reads at appropriate layer (per-user/in-memory/Redis) with TTL and invalidation hooks.
- **Follow-up question:** How would you detect accidental sync calls inside async endpoints in production?

### 3. Global exception handling with consistent JSON
- **Design:** Add a global exception handler using FastAPI’s exception handlers and middleware to normalize error shape.
- **Structure:** Return a standard payload: { error_code, message, details, correlation_id, timestamp } and map framework/DB errors to domain codes.
- **Security:** Avoid leaking internals; log root causes with structured logging; expose safe messages to clients.
- **Traceability:** Include correlation/request IDs to link logs and traces; integrate with APM.
- **Follow-up question:** How would you version your error codes so clients can reliably handle changes over time?

### 4. JWT auth with role-based access
- **Flow:** Issue short-lived access tokens and refresh tokens; store minimal claims (sub, roles, exp) and validate via public keys (JWKS).
- **RBAC:** Implement role checks via dependencies and route-level decorators; use policy objects or ABAC for complex rules.
- **Security hardening:** Use rotating keys, audience/issuer checks, clock skew tolerance, and token revocation lists for high-risk operations.
- **Multi-tenancy:** Include tenant_id claim and enforce RLS at the DB layer when applicable.
- **Follow-up question:** When would you choose token introspection over self-contained JWTs?

### 5. Project structure for microservices
- **Organization:** Modularize by bounded contexts; each service has its own repo, CI/CD, schema migrations, and versioned contracts.
- **Shared components:** Common libs for auth, tracing, error models; avoid shared DBs—communicate via APIs/events.
- **Data ownership:** Per-service databases; use CDC/event bus for cross-service sync; define SLOs per service.
- **Dev workflows:** Scaffold via templates, enforce linting/tests, and maintain API docs via OpenAPI.
- **Follow-up question:** How would you manage cross-cutting schema changes that touch multiple services without creating a lockstep release?

### 6. Debug latency spikes in Kubernetes
- **Stepwise isolation:** Check pod resource saturation (CPU/memory), event-loop lag, GC pauses; compare intra-pod vs cluster network latency.
- **DB path:** Inspect connection pool wait time, slow query logs, locks, and replication lag; run EXPLAIN ANALYZE on hot paths.
- **Network:** Examine CNI, service mesh overhead, DNS resolution, and MTU issues; capture p99 and tail latencies.
- **Tools:** Use pprof/py-spy, APM traces across app→DB, and kube events for rescheduling or throttling.
- **Follow-up question:** What signals would make you suspect a network path problem over an application-level issue?

### 7. Rate limiting specific endpoints
- **Techniques:** Token bucket or leaky bucket per user/IP/key; implement via middleware or API gateway (e.g., Envoy/NGINX Ingress).
- **Storage:** Use Redis for counters with sliding window or fixed window with jitter; include headers (X-RateLimit-*) in responses.
- **Fairness:** Different tiers per endpoint and per role; graceful 429 with retry-after hints.
- **Abuse detection:** Add anomaly detection and ban lists; exempt health and critical system flows.
- **Follow-up question:** How would you prevent Redis hot keys when a single endpoint sees extreme burst traffic?

### 8. Background tasks without blocking
- **Pattern:** Offload work to task queues (Celery, RQ, Dramatiq) or FastAPI BackgroundTasks for lightweight, in-process tasks.
- **Reliability:** Use durable queues, retries with backoff, idempotency keys, and outbox pattern for DB-triggered events.
- **Response:** Return immediately with task ID; expose status endpoints and webhooks; consider Sagas for multi-step processes.
- **Isolation:** Avoid long CPU tasks in API pods; separate worker autoscaling from API autoscaling.
- **Follow-up question:** How would you guarantee exactly-once effects for an email send triggered by a DB write?

---

# PostgreSQL scenarios

### 9. Optimize 50M-row table with EXPLAIN ANALYZE
- **Process:** Capture slow query; run EXPLAIN (ANALYZE, BUFFERS) to see plan, node timings, and IO; use auto_explain in prod with thresholds.
- **Focus areas:** Check index usage, seq scans, hash vs nested loops, join order, filter selectivity, and row estimates vs actuals.
- **Actions:** Create appropriate indexes, rewrite queries, add partial indexes, adjust work_mem/JIT, or materialize heavy subqueries.
- **Data layout:** Consider partitioning by time or tenant; vacuum/analyze to fix bloat and stats.
- **Follow-up question:** When would you choose to rewrite a query to avoid a nested loop rather than increase work_mem?

### 10. Indexes vs materialized views vs partitions
- **Indexes:** Best for targeted lookups and selective predicates; maintainable but add write overhead.
- **Materialized views:** Precompute heavy aggregations/reports; refresh on schedule or with CONCURRENTLY; great for read-heavy dashboards.
- **Partitions:** Manage very large tables/time-series; improve pruning and maintenance; complexity in planning/constraints.
- **Decision framework:** Balance write impact, freshness needs, and query patterns; sometimes combine (partition + indexes + matview).
- **Follow-up question:** How would you handle staleness in a materialized view without blocking writes?

### 11. Handle deadlocks and race conditions
- **Prevention:** Access resources in a consistent order and keep transactions short; avoid unnecessary locks.
- **Detection:** Monitor pg_locks and logs; set deadlock_timeout sensibly; capture wait graphs.
- **Resolution:** Use retries with jitter on serialization failures; move contentious updates to queues or use optimistic concurrency.
- **Design:** Consider application-level fences or version checks; break hot rows by sharding counters.
- **Follow-up question:** What patterns help minimize contention on “hot” rows like global counters?

### 12. Optimistic vs pessimistic locking
- **Optimistic:** Use version columns (xmin or custom) and WHERE version=… on update; fail fast on conflicts; ideal for low contention.
- **Pessimistic:** SELECT … FOR UPDATE/SHARE to block concurrent writers; suitable for high-stakes resources but can reduce throughput.
- **Trade-offs:** Optimistic improves concurrency and requires retry logic; pessimistic simplifies correctness with potential deadlocks.
- **Hybrid:** Combine coarse-grained pessimistic with fine-grained optimistic for complex workflows.
- **Follow-up question:** When would you prefer SKIP LOCKED over NOWAIT for queue consumption?

### 13. Event logging schema for massive writes
- **Model:** Append-only table with JSONB payload, indexed on key dimensions (time, type, tenant, user); write via batch inserts.
- **Performance:** Partition by time (daily/hourly), use UNLOGGED or buffered writes where acceptable; minimize constraints/triggers.
- **Querying:** Create partial indexes for common filters; materialize aggregates; keep payload normalized as needed for analytics.
- **Retention:** Tier storage and implement TTL/archival; compress old partitions.
- **Follow-up question:** How would you design indexes to support both time-range queries and top-N aggregations efficiently?

### 14. Read replicas, sharding, or caching
- **Replicas:** Offload reads; consider lag and read-your-writes semantics; route with client or proxy.
- **Sharding:** Split by tenant or key when a single node limits CPU/storage; increases operational complexity.
- **Caching:** Use Redis/memcached for hot queries; tie invalidation to write paths or outbox events.
- **Approach:** Start with caching and replicas; shard only when necessary and clearly justified by data distribution.
- **Follow-up question:** How would you guarantee correctness for a flow that requires read-after-write on a system with replicas?

### 15. Zero-downtime schema migrations
- **Pattern:** Backward-compatible migration steps: add nullable columns, deploy code that writes both, backfill asynchronously, flip reads, then enforce constraints.
- **Tools:** Alembic with online migrations; use transactional DDL where possible; avoid locks via CONCURRENTLY operations.
- **Safety:** Feature flags, canaries, and rollbacks; ensure migration idempotency and thorough testing.
- **Coordination:** Communicate sequencing across services; freeze high-risk operations during critical windows.
- **Follow-up question:** How would you handle backfills that exceed maintenance windows without impacting OLTP?

### 16. Multi-tenant schema design
- **Options:** Shared schema with tenant_id and RLS; schema-per-tenant; database-per-tenant for isolation.
- **Trade-offs:** Shared schema scales better and is manageable; schema-per-tenant eases noisy-neighbor issues but complicates migrations.
- **Security:** Enforce RLS with session variables and policies; audit tenant-bound access.
- **Operations:** Consider partitioning by tenant or time; plan for per-tenant export/backup.
- **Follow-up question:** What signals would make you move from shared schema to schema-per-tenant?

---

# FastAPI + PostgreSQL combined scenarios

### 17. Fix asyncpg connection pool exhaustion
- **Symptoms:** Long waits for a connection, timeouts, increased latency; pool size too small or connections leaked.
- **Debugging:** Instrument acquire/release; track in-flight queries; set pool max_size/min_size; verify release in finally blocks.
- **Solutions:** Increase pool size within DB limits; limit concurrency per endpoint; implement query timeouts and cancellation; reuse prepared statements.
- **Guardrails:** Detect slow consumers; add circuit breakers; monitor pg_stat_activity for idle-in-transaction sessions.
- **Follow-up question:** How would you detect and prevent “idle in transaction” states from your FastAPI code?

### 18. Zero-downtime Alembic migrations for new column with constraints
- **Steps:** Add column nullable; deploy code writing both old and new; backfill in batches; add index/constraint CONCURRENTLY; switch reads; make column NOT NULL.
- **Safety:** Use feature flags; retryable batches; lock-aware operations; ensure default values are safe.
- **Rollback:** Keep dual-write until stable; scripts to revert reads; avoid destructive changes until validated.
- **Testing:** Dry run with production-like data; verify scan times; observe locks and replication impact.
- **Follow-up question:** How would you ensure backfill jobs don’t overwhelm the connection pool or I/O?

### 19. Caching heavy report with Redis + middleware, ensure consistency
- **Strategy:** Cache rendered report keyed by parameters; short TTL or write-through invalidation on data changes.
- **Consistency:** Use cache tags or sets per entity; invalidate on outbox events; prefer stale-while-revalidate for UX.
- **Controls:** Include ETag and If-None-Match; expose cache hit/miss metrics; avoid caching per-user secrets.
- **Fallbacks:** Provide degraded summaries when cache cold; pre-warm for known periods.
- **Follow-up question:** How would you design cache invalidation for a report joining multiple large tables?

### 20. Pagination for large datasets
- **Approach:** Use keyset pagination (WHERE x > last_value ORDER BY x ASC LIMIT n) for stable performance; avoid OFFSET for deep pages.
- **Consistency:** Tie sort to indexed, unique columns; include next cursors; handle backward pagination via inverse keyset.
- **API design:** Return metadata (next, prev, total if cheap); avoid computing total counts on hot paths.
- **DB side:** Ensure covering indexes and selective filters; guard against gaps due to deletes.
- **Follow-up question:** When is OFFSET acceptable, and how would you mitigate its performance impact?

### 21. Audit logging for API requests and DB changes
- **API logs:** Middleware capturing actor, route, params (sanitized), response code, correlation_id, latency.
- **DB logs:** Outbox/audit tables with operation, before/after, actor, timestamp; use triggers sparingly, prefer app-level logs for context.
- **Storage:** Write-ahead to durable queue; batch to cold storage; partition and retain.
- **Integrity:** Hash chains or signatures for compliance; protect against tampering.
- **Follow-up question:** How would you reconcile API-layer audit logs with DB-layer change logs to prove end-to-end causality?

### 22. Ensure transactional consistency
- **Pattern:** Wrap business operations in explicit transactions; use SERIALIZABLE/REPEATABLE READ where needed; leverage sagas for cross-service flows.
- **Idempotency:** Assign operation IDs; upsert with unique constraints to avoid duplicates.
- **Outbox:** Persist events in same transaction, publish asynchronously; guarantee exactly-once via de-duplication.
- **Timeouts:** Bound transaction duration; avoid long-lived locks; check for retries on conflicts.
- **Follow-up question:** When would you choose SERIALIZABLE isolation, and how would you handle its retry burden?

### 23. Monitor and trace slow queries with Prometheus/Grafana
- **Metrics:** Export handler latencies, pool stats, error rates; from DB, track slow query count, locks, buffer hits, cache ratios.
- **Tracing:** Use OpenTelemetry to instrument app→DB spans; annotate with SQL signatures and bind variables anonymized.
- **Dashboards:** Correlate p99 app latency with DB wait time and network; alert on threshold breaches.
- **Workflows:** Sentry for errors, logs with correlation IDs; run bloat/index scans periodically.
- **Follow-up question:** How would you design exemplars or trace sampling to capture just enough detail without high overhead?

---

# FastAPI advanced scenarios

### 24. Add GraphQL alongside REST
- **Design:** Introduce a GraphQL endpoint under a separate path; reuse service layer; keep REST contracts stable.
- **Schema:** Model types and resolvers carefully; avoid N+1 via dataloaders; enforce auth/roles consistently.
- **Performance:** Limit query depth/complexity; cache resolvers; paginate connections.
- **Docs & adoption:** Provide clear migration guides; maintain both interfaces; deprecate gradually if needed.
- **Follow-up question:** How would you prevent overly expensive GraphQL queries from degrading REST performance?

### 25. WebSockets for real-time notifications at scale
- **Scaling:** Use a pub/sub backend (Redis, NATS) with per-instance subscription; sticky sessions where necessary; offload fan-out to a broker.
- **Lifecycle:** Heartbeats and pings; backpressure on message queues; disconnect idle clients.
- **Security:** Auth during handshake; per-tenant channels; rate limit messages.
- **Operations:** Horizontal scale with connection limits per pod; monitor memory and descriptor usage.
- **Follow-up question:** How would you deliver ordered messages across multiple instances?

### 26. Dependency injection for services
- **Pattern:** Use FastAPI’s Depends with provider functions for DB session, cache clients, and auth; manage lifetimes (request vs app).
- **Testing:** Provide test doubles via overrides; inject configuration contexts.
- **Performance:** Avoid expensive object creation per request; use connection pools and lazy init.
- **Boundaries:** Keep DI at controller layer; services pure with minimal framework coupling.
- **Follow-up question:** How would you handle per-request tenants in DI without leaking state across requests?

### 27. Localization middleware for multi-language errors
- **Design:** Parse Accept-Language or user preference; load translations (i18n catalogs); map error codes to localized messages.
- **Middleware:** Attach locale to request context; format responses consistently; fallback hierarchy (e.g., Tamil → English).
- **Storage:** Keep translation keys stable; handle pluralization and interpolation safely.
- **Testing:** Snapshot error outputs per locale; ensure logging stays in a standard language.
- **Follow-up question:** How would you manage runtime updates to translation catalogs without restarting services?

### 28. API versioning with backward compatibility
- **Approach:** Path-based (/v1, /v2) or header-based; maintain old versions until deprecation window; use adapters.
- **Compatibility:** Keep non-breaking additions; introduce new fields behind feature flags; provide migration docs.
- **Data contracts:** Use schema evolution in DTOs; ensure clients can ignore unknown fields.
- **Governance:** Version policies, sunset timelines, and telemetry to gauge usage.
- **Follow-up question:** How would you version long-running streaming endpoints or WebSockets?

### 29. Request validation for nested JSON with Pydantic
- **Models:** Compose nested BaseModels with validators; use constrained types and enums; leverage root validators for cross-field rules.
- **Performance:** Use pydantic v2 where available for speed; pre-parse large lists with strict types.
- **Errors:** Return structured error messages; map to localization if needed.
- **Maintainability:** Split models per endpoint and domain; reuse where stable.
- **Follow-up question:** How would you validate conditional fields (e.g., one-of semantics) elegantly?

### 30. Integrate Celery/background workers for long tasks
- **Architecture:** API enqueues tasks with idempotency keys; workers process with retries/backoff; results stored in a durable store.
- **Communication:** Use result backend for status; webhooks or polling; avoid returning large payloads over brokers.
- **Observability:** Track task metrics, queues, failure causes; circuit-break problematic tasks.
- **Scaling:** Autoscale workers; isolate queues by priority; prevent noisy neighbors.
- **Follow-up question:** How would you ensure tasks are not duplicated when the API retried enqueue on network errors?

### 31. Graceful shutdown in Kubernetes
- **Mechanics:** Handle SIGTERM; stop accepting new requests; finish in-flight with a timeout; close pools; drain WebSockets.
- **Coordination:** Use preStop hooks and terminationGracePeriodSeconds; readiness gates to remove from load balancer first.
- **Idempotency:** Persist checkpoints for background work; ensure workers requeue unfinished tasks.
- **Validation:** Test with rolling updates and chaos experiments; verify no 5xx spikes.
- **Follow-up question:** How would you handle draining for long-lived streams that may exceed termination grace periods?

---

# PostgreSQL advanced scenarios

### 32. JSON vs JSONB and indexing strategies
- **Choice:** JSONB for most cases—binary format, supports indexing and efficient operators; JSON if exact text preservation matters.
- **Indexes:** GIN indexes on JSONB with appropriate operator classes; partial indexes for common paths; expression indexes.
- **Queries:** Avoid deep traversal per row; project needed fields; combine with materialized computed columns if hot paths.
- **Maintenance:** Monitor index size and bloat; keep statistics updated.
- **Follow-up question:** When would you prefer a normalized table over JSONB despite flexible schemas?

### 33. Full-text search for a blog
- **Setup:** Use tsvector columns with language-specific dictionaries; create GIN indexes; maintain via triggers or generated columns.
- **Features:** Phrase search, ranking with ts_rank, highlighting; combine with filters (tags, author, date).
- **Scaling:** Precompute tsvectors; use materialized views for heavy aggregations; shard only if necessary.
- **Relevance:** Adjust weights per field (title > body > tags).
- **Follow-up question:** How would you handle multilingual posts in FTS without losing ranking quality?

### 34. Multi-region PostgreSQL replication
- **Approach:** Use physical streaming replicas per region; or logical replication for selective tables; consider managed solutions.
- **Consistency:** Prefer primary-region writes; read-local with awareness of lag; achieve global writes via CRDTs or app-level conflict resolution.
- **Latency:** Route clients to nearest read region; cache aggressively; batch cross-region writes.
- **Failover:** Orchestrate controlled failover; DNS and app config updates; test regularly.
- **Follow-up question:** What application patterns reduce the pain of eventual consistency across regions?

### 35. Partitioning for time-series billions of readings
- **Strategy:** Range partitions by time (daily/hourly); attach indexes per partition; use partition pruning.
- **Ingest:** Batch inserts, COPY, and unlogged staging tables; move/merge partitions off-peak.
- **Querying:** Prefer aggregate rollups, hypertables (TimescaleDB) if available; keep hot partitions small.
- **Maintenance:** Automatic partition creation; retention and compression policies for old data.
- **Follow-up question:** How would you handle out-of-order late-arriving events with strict time partitions?

### 36. Row-level security for multi-tenant
- **Implementation:** Enable RLS; set app_user/tenant via SET LOCAL or parameter; policies filtering on tenant_id.
- **Safety:** Deny by default; explicit policies for reads/writes; test thoroughly with EXPLAIN and role simulations.
- **Performance:** Index tenant_id; ensure partitioning aligns with tenant filters when needed.
- **Auditing:** Log policy hits/misses; detect bypass attempts.
- **Follow-up question:** How would you avoid RLS-related surprises during admin/maintenance operations?

### 37. Schema evolution with shared DB among microservices
- **Governance:** Contract-first changes; backward-compatible migrations; change advisory boards and deprecation windows.
- **Techniques:** Additive changes first; dual-write/read; phased removals; feature flags.
- **Isolation:** Prefer separate schemas per service; restrict cross-service tables; use views to present stable contracts.
- **Testing:** Consumer-driven contracts; shadow reads; canary deployments.
- **Follow-up question:** How would you handle a breaking change if one service cannot update in time?

### 38. Backup and disaster recovery strategy
- **Backups:** Regular base backups with WAL archiving; verify restore via periodic drills; encrypt at rest.
- **RPO/RTO:** Define tiers; consider point-in-time recovery; geo-redundant storage.
- **Automation:** Use tools for scheduled backups and integrity checks; monitor backup lag and failures.
- **Failover:** Runbooks for promotion; test chaos scenarios and corrupted data cases.
- **Follow-up question:** How would you protect against silent data corruption that passes backup integrity checks?

### 39. Use of CTEs to simplify complex queries
- **Clarity:** Break multi-step logic into named CTEs; improve readability and maintainability.
- **Performance:** Beware of optimization fences (older versions); in modern PostgreSQL, CTEs are often inlined—still measure.
- **Patterns:** Use CTEs for recursive queries (hierarchies) and staged aggregations.
- **Alternative:** Materialize heavy intermediate results when beneficial; consider temp tables.
- **Follow-up question:** When would you replace a CTE pipeline with a materialized view?

---

# FastAPI + PostgreSQL combined scenarios (second set)

### 40. Optimize bulk inserts
- **Methods:** Use COPY or executemany with batches; disable unnecessary constraints/triggers; use UNLOGGED staging tables.
- **Transactions:** Batch size tuned to memory and lock duration; commit per batch; use prepared statements.
- **Indexes:** Delay index creation until after load or use minimal necessary indexes; vacuum/analyze after.
- **Integrity:** Validate input offline; deduplicate via unique indexes in final table.
- **Follow-up question:** How would you preserve idempotency for bulk upserts at scale?

### 41. Transaction management across multiple operations
- **Design:** Use explicit transactions in the service layer; map business steps to DB operations; rollback on failure.
- **Isolation:** Choose appropriate levels; avoid user-level waits; use savepoints for partial retries.
- **Sagas:** For cross-resource workflows, coordinate with compensating actions; log transitions.
- **Observability:** Emit events and metrics for transaction durations and failures.
- **Follow-up question:** What’s your approach to ensure compensating actions don’t cause cascading failures?

### 42. Connection pooling parameters
- **Tuning:** Set max pool size based on DB max_connections minus overhead; min_size to keep warm; timeout for acquisition.
- **Per endpoint:** Cap concurrency via semaphores; timeouts per query; cancel on client disconnect.
- **Health:** Track pool waits, queue lengths, and rate of timeouts; use circuit breaking on exhaustion.
- **DB side:** Use pgbouncer in transaction mode to increase effective concurrency.
- **Follow-up question:** How would you set different pool sizes for read-heavy vs write-heavy workloads?

### 43. Multi-tenancy design choice
- **Separate schemas:** Better isolation, easier per-tenant maintenance; more complex migrations.
- **Shared schema:** Simpler operations, better resource sharing; enforce with RLS and tenant_id.
- **Decision drivers:** Tenant count, data size variance, compliance needs, noisy neighbor risk.
- **Hybrid:** VIP tenants in separate schemas; others in shared.
- **Follow-up question:** How would you plan a migration from shared to per-tenant schemas with minimal downtime?

### 44. Audit logging for every call and transaction
- **API layer:** Middleware captures request metadata and result; ensure PII redaction.
- **DB layer:** Outbox/audit tables for CRUD with actor and diff; triggers only for critical paths.
- **Correlation:** Use correlation_id end-to-end; join logs for forensic trails.
- **Storage:** Partition, compress, and retain based on policy.
- **Follow-up question:** How would you detect and eliminate duplicate audit entries from retries?

### 45. Soft deletes in PostgreSQL and FastAPI
- **Schema:** Add deleted_at or is_deleted; index for filtering; enforce checks in queries/views.
- **API:** Expose restore endpoints; default queries exclude soft-deleted; provide “include_deleted” flags.
- **Integrity:** Unique constraints may need partial indexes to ignore deleted rows.
- **Compliance:** Periodic hard-delete/archival jobs; audit deletes and restores.
- **Follow-up question:** How would you guarantee unique constraints while allowing soft deletes?

### 46. Reporting API without blocking event loop
- **Pattern:** Offload heavy compute to workers; precompute aggregates; stream responses via server-side cursors.
- **Async:** Use database cursors and chunked transfer; limit per-request CPU; paginate reports.
- **Caching:** Cache results; expose applied filters in metadata; allow async job with callback.
- **Protection:** Enforce query complexity limits and timeouts.
- **Follow-up question:** When would you choose to return a job handle instead of synchronous report data?

### 47. Integrate LISTEN/NOTIFY for real-time updates
- **Mechanism:** Open a dedicated async connection for LISTEN; push notifications to WebSockets or SSE.
- **Reliability:** Buffer events for reconnects; confirm delivery with ack or pull-based reconciliation.
- **Security:** Filter per tenant; avoid leaking payloads; throttle noisy channels.
- **Scale:** Combine with pub/sub; use channels per topic; monitor notify queue length.
- **Follow-up question:** How would you handle missed notifications during a restart to avoid stale client views?

---

# FastAPI challenge-oriented questions

### 48. Performance bottlenecks under high load
- **Root cause:** Blocking operations inside async handlers (e.g., sync file I/O); small connection pool; N+1 DB queries.
- **Fix:** Refactor to async, batch queries, increase pool, introduce caching, and add circuit breakers; load test iteratively.
- **Outcome:** Reduced p99 latency and stabilized throughput; fewer timeouts.
- **Learning:** Always measure event-loop lag and pool wait times.
- **Follow-up question:** What early warning metrics would have caught this before users noticed?

### 49. Async/await misuse
- **Issue:** Missing awaits and accidental sync calls caused hidden deadlocks and starvation.
- **Debug:** Traces showed long spans; used asyncio debug mode and profiling; identified blocking libraries.
- **Resolution:** Replaced libraries with async equivalents; moved CPU tasks to workers; added lint rules.
- **Prevention:** Code reviews focusing on async purity; unit tests simulating concurrency.
- **Follow-up question:** What static analysis or CI checks would you add to flag async antipatterns?

### 50. Dependency injection unexpected behavior
- **Problem:** Shared mutable singletons leaked state across requests (tenant/user context).
- **Fix:** Scoped dependencies to request; used contextvars; avoided global caches for request-bound data.
- **Outcome:** Eliminated cross-request contamination; improved security and correctness.
- **Follow-up question:** How would you test for hidden shared-state issues under concurrency?

### 51. JWT auth challenges (refresh/expiry)
- **Issue:** Clock skew and premature expiry; refresh token rotation not handled correctly.
- **Fix:** Added leeway for verification; rotate refresh tokens atomically; tracked revocation; short-lived access tokens.
- **Result:** Secure flow with minimal user impact; reduced token-related 401s.
- **Follow-up question:** How would you detect refresh token theft and mitigate blast radius?

### 52. CORS errors with frontend integration
- **Cause:** Mismatched origins and missing credentials flags; preflight blocked; wrong headers.
- **Fix:** Configured allowed origins/methods/headers; enabled credentials when needed; handled preflight OPTIONS.
- **Validation:** Tested via curl and browser devtools; added strict but explicit rules.
- **Follow-up question:** How would you safely support dynamic origins for multi-tenant frontends?

### 53. Background tasks reliability bug
- **Issue:** Tasks failed silently on worker restarts; missing idempotency; retries caused duplicates.
- **Fix:** Added durable queues, task status store, idempotency keys, and alerting; implemented at-least-once with deduplication.
- **Impact:** No lost tasks; consistent outcomes; transparent monitoring.
- **Follow-up question:** How would you design compensating actions for tasks with external side effects?

### 54. Pydantic model validation for complex JSON
- **Issue:** Incorrect nested defaults and permissive types; cross-field constraints unmet.
- **Fix:** Strict types, custom validators, union models for variants, and one-of validation patterns.
- **Outcome:** Clear error messages; fewer runtime errors; better client contracts.
- **Follow-up question:** How would you keep validation fast for very large payloads?

### 55. API versioning challenges
- **Issue:** Breaking changes needed while legacy clients depended on old payload shapes.
- **Fix:** Introduced v2 alongside v1 with adapters; telemetry-guided deprecation; contract testing with clients.
- **Result:** Smooth migration and adoption; minimal outages.
- **Follow-up question:** How would you decide when to fully retire a legacy version?

### 56. WebSocket scaling issues
- **Issue:** Single instance handling too many connections; memory pressure; unordered broadcasts.
- **Fix:** Moved to broker-backed pub/sub, sharded channels, and sticky sessions; backpressure and batching.
- **Result:** Stable connections and ordered delivery guarantees.
- **Follow-up question:** How would you detect memory leaks caused by lingering WebSocket references?

### 57. Difficult deployment bug (Kubernetes/Docker)
- **Issue:** Latency spikes due to DNS resolution failures and misconfigured readiness probes causing thrash.
- **Fix:** Tuned DNS caching, probe thresholds, and resource requests; added circuit breakers.
- **Outcome:** Reduced restarts and stabilized latency.
- **Follow-up question:** What chaos tests would you run to validate resilience post-fix?

---

# PostgreSQL challenge-oriented questions

### 58. Very slow PostgreSQL query
- **Bottleneck:** Seq scan with misestimated row counts; missing composite index; join order suboptimal.
- **Fix:** Added selective composite index; adjusted statistics target; rewrote joins; analyzed tables.
- **Result:** Orders-of-magnitude improvement in latency and fewer IO spikes.
- **Follow-up question:** How would you guard against stats drift causing performance regressions later?

### 59. Deadlocks/race conditions in transactions
- **Scenario:** Two workflows updating shared resources in different orders.
- **Resolution:** Standardized lock acquisition order; reduced transaction scope; added retries on deadlocks.
- **Outcome:** Eliminated deadlocks and improved throughput.
- **Follow-up question:** How would you simulate deadlocks reliably in tests?

### 60. Connection pooling challenges
- **Issue:** Exhaustion due to long queries and leaked sessions.
- **Fix:** Timeouts, cancellation on client disconnect, strict acquire/release patterns, and pgbouncer.
- **Monitoring:** Alerting on pool wait times and idle-in-transaction count.
- **Follow-up question:** What metrics indicate you should increase pool size vs optimize queries?

### 61. Schema migration problems in production
- **Issue:** Locks during index creation blocked writes.
- **Fix:** Used CONCURRENTLY options; split migrations; performed backfills off-peak; feature flags.
- **Outcome:** No downtime and predictable rollouts.
- **Follow-up question:** How would you recover if a concurrent index build fails halfway?

### 62. Index bloat/wrong indexing
- **Problem:** Over-indexing and low-selectivity indexes; bloated GIN.
- **Fix:** Dropped redundant indexes; used partial indexes; reindexed; tuned vacuum.
- **Result:** Lower write amplification and improved queries.
- **Follow-up question:** How do you decide when a covering index is worth the write cost?

### 63. JSON/JSONB field bug
- **Issue:** Deep path queries slow without expression indexes; inconsistent schemas in payloads.
- **Fix:** Added generated columns for hot paths and indexed them; enforced schema at ingestion.
- **Outcome:** Stable performance and simpler queries.
- **Follow-up question:** How would you validate JSON structure at the database boundary?

### 64. Partitioning large tables challenges
- **Issue:** Poor pruning and hotspot partitions.
- **Fix:** Aligned partition keys with queries; created smaller, regular partitions; moved hot writes to separate tables.
- **Outcome:** Faster scans and predictable maintenance.
- **Follow-up question:** How would you monitor partition pruning effectiveness over time?

### 65. Replication lag
- **Cause:** Heavy writes and long transactions; network latency.
- **Fix:** Reduced batch sizes, optimized queries, tuned wal settings; added read routing with lag awareness.
- **Outcome:** Acceptable lag and fewer stale reads.
- **Follow-up question:** What client-side strategies mitigate read inconsistency due to lag?

---

# Combined FastAPI + PostgreSQL challenge questions

### 66. Async queries caused connection leaks
- **Cause:** Exceptions before release; missing finally blocks; improper session scoping.
- **Fix:** Context managers for sessions; middleware to ensure cleanup; pool instrumentation and alerts.
- **Result:** No leaks and stable latency under load.
- **Follow-up question:** How would you detect rare leaks that only occur under specific error paths?

### 67. Alembic migrations broke endpoints
- **Issue:** Code assumed presence of new column before migration completed on all nodes.
- **Fix:** Backward-compatible code, feature flags, and staged deploys; health checks on schema state.
- **Outcome:** Resilient rollouts even with staggered nodes.
- **Follow-up question:** How would you gate endpoint availability based on runtime schema checks?

### 68. Data inconsistency between logic and transactions
- **Issue:** Side effects outside transactions led to partial updates.
- **Fix:** Wrap all mutations; use outbox for external events; enforce idempotency with unique keys.
- **Outcome:** Stronger guarantees and recoverability.
- **Follow-up question:** How would you reconcile when a downstream consumer processed an event twice?

### 69. Caching stale PostgreSQL queries in FastAPI
- **Issue:** Cache didn’t invalidate on updates; users saw old data.
- **Fix:** Event-driven invalidation from outbox; short TTLs; stale-while-revalidate.
- **Outcome:** Freshness with performance balance.
- **Follow-up question:** How would you handle cache stampede when a popular key expires?

### 70. Pagination returned wrong results
- **Issue:** Offset-based pagination with concurrent inserts caused duplicates/misses.
- **Fix:** Switched to keyset pagination with stable sort key; careful filtering.
- **Outcome:** Correct and performant pagination.
- **Follow-up question:** How would you handle pagination across joined tables with composite sort keys?

### 71. Audit logging missing/duplicate logs
- **Issue:** Retries and partial failures led to mismatched logs.
- **Fix:** Transactional outbox for DB changes; idempotent API logging; dedupe by correlation + operation IDs.
- **Outcome:** Consistent, reconcilable audit trails.
- **Follow-up question:** How would you verify audit completeness daily?

### 72. Memory leaks due to DB session handling
- **Issue:** Long-lived sessions retained references; unclosed cursors; caches growing unbounded.
- **Fix:** Scoped sessions to requests; periodic memory profiling; caps on caches; contextvars cleanup.
- **Outcome:** Stable memory footprint over time.
- **Follow-up question:** What production signals would trigger a memory leak investigation?

---

# Final set: FastAPI + PostgreSQL combined scenarios

### 73. Bulk inserts with FastAPI
- **Flow:** Accept uploads, validate asynchronously, write via COPY/executemany in batches; stream responses.
- **Backpressure:** Limit concurrent bulk jobs; queue tasks; return job IDs.
- **Safety:** Idempotency with content hashes; partial failure reporting.
- **Monitoring:** Throughput, error rates, and DB impact; pause on overload.
- **Follow-up question:** How would you coordinate bulk upload windows to avoid starving OLTP traffic?

### 74. Transaction management in FastAPI
- **Implementation:** Dependency provides transactional session; unit-of-work pattern; explicit commit/rollback.
- **Error handling:** Map exceptions to retries or user errors; timeouts to avoid long locks.
- **Idempotency:** Ensure safe retries; unique constraints for dedupe.
- **Tracing:** Emit spans for transaction boundaries.
- **Follow-up question:** How would you expose partial success vs full failure semantics to clients?

### 75. Connection pooling configuration
- **Parameters:** max_size, min_size, max_queries per connection, max_inactive_time; acquisition and query timeouts.
- **Tuning:** Base on workload and DB limits; profile under load; isolate read pools vs write pools.
- **Protections:** Cancel long queries on disconnect; enforce per-route concurrency caps.
- **Observability:** Export pool metrics; alert on saturation.
- **Follow-up question:** How would you safely change pool size at runtime?

### 76. Multi-tenancy schema vs shared
- **Decision:** Shared with RLS for many small tenants; schema-per-tenant for large or regulated tenants.
- **Operations:** Separate migrations pipelines per schema; tenant onboarding automation.
- **Performance:** Partitioning and indexing by tenant; query guards.
- **Security:** Strict policy enforcement and audit.
- **Follow-up question:** How would you run per-tenant backups and restores efficiently?

### 77. Audit logging for API and DB
- **Unified approach:** Middleware + DB outbox; join via correlation IDs; immutable store.
- **Indexes:** On actor, tenant, resource; partition by time.
- **Review:** Dashboards and anomaly detection; reconciliation jobs.
- **Compliance:** Sign logs or chain hashes.
- **Follow-up question:** How would you prove to auditors that logs weren’t tampered with?

### 78. Soft deletes API design
- **Contract:** DELETE marks deleted_at; GET excludes unless include_deleted=true; RESTORE clears deleted_at.
- **Consistency:** Unique partial indexes; cascade soft deletes via business rules.
- **Visibility:** Admin endpoints to view/manage deleted records.
- **Monitoring:** Track delete/restore rates and anomalies.
- **Follow-up question:** How would you prevent accidental permanent loss during batch purges?

### 79. Reporting API for large datasets
- **Design:** Async job creation; precomputations; streaming cursors; pagination.
- **Protection:** Query limits and timeouts; cache outputs; cost-based controls.
- **UX:** Polling with progress; webhooks when ready; ETag for re-downloads.
- **Infra:** Separate read replicas for heavy reporting.
- **Follow-up question:** How would you estimate and communicate expected report time to users?

### 80. LISTEN/NOTIFY integration
- **Flow:** DB emits NOTIFY on changes; FastAPI subscriber relays to clients; reconcile with periodic pulls.
- **Resilience:** Buffer during downtime; de-dup with message IDs; handle bursts with backpressure.
- **Security:** Tenant-scoped channels; validate permissions.
- **Scale:** Consider external brokers if volume exceeds Postgres notify capabilities.
- **Follow-up question:** How would you migrate from DB notifications to a dedicated event bus with minimal disruption?