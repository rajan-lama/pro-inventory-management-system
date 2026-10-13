# IMPLEMENTATION_PLAN.md — Delivery Roadmap

## Working method
Deliver a vertical slice in every phase. Keep a requirements checklist and architecture decision records. Do not start all modules in parallel before the domain contracts stabilize.

## Phase 0 — Discovery and decisions
Deliverables:
- Confirm business types, users, branch/warehouse model, accounting scope, offline requirements, payment methods, costing method, approval thresholds, expected scale.
- Verify current IRD/CBMS requirements and API access.
- Define terminology, invoice lifecycle, returns/cancellation process, fiscal-year policy, tax calculation fixtures.
- Write initial ADRs for modular monolith, tenancy strategy, auth strategy, stock ledger, numbering, and outbox.

Exit criteria:
- Client signs off on workflow diagrams and unresolved compliance risks are recorded.

## Phase 1 — Foundation, identity, tenancy
Deliverables:
- Spring Boot project structure, PostgreSQL, Flyway, CI, test setup, OpenAPI.
- Users, organizations, memberships, branches, warehouses.
- Roles, permissions, invitations, account lifecycle, audit/security events.
- Tenant context, server-side authorization, scope restrictions.
- Basic admin UI/API and tenant-isolation tests.

Exit criteria:
- A user can belong to two organizations with different permissions.
- Cross-tenant and unauthorized branch/warehouse access tests pass.

## Phase 2 — Catalog and inventory ledger
Deliverables:
- Products, categories, units, SKUs/barcodes, tax classification fields.
- Stock ledger and balance projection.
- Opening balances, goods receipt, adjustment, count, reservation, transfer.
- Costing method decision and initial valuation.
- Reconciliation report.

Exit criteria:
- Ledger-to-balance reconciliation passes.
- Concurrent reservations and transfers are safe.
- Every correction is traceable.

## Phase 3 — Purchasing
Deliverables:
- Suppliers, purchase orders, goods receipts, supplier bills, purchase returns.
- Partial receipt and landed-cost behavior.
- Supplier balances and purchase register.

Exit criteria:
- Purchase order alone does not increase stock.
- Receipts and returns generate the correct ledger movements.

## Phase 4 — Sales, POS, billing
Deliverables:
- Customers, quotations/orders if required, POS checkout.
- Draft/final invoice lifecycle, transactional numbering, invoice PDF/print.
- Tax engine and historical tax snapshots.
- Payment records, partial settlement, returns, credit notes, refunds.
- Reprint and audit workflows.

Exit criteria:
- Idempotent checkout.
- Concurrency tests pass for invoice numbering.
- Financial documents and stock movements reconcile.

## Phase 5 — IRD/CBMS and statutory reports
Deliverables:
- Verified adapter and API fixtures.
- Durable outbox and submission attempt store.
- Retry/reconciliation workflow and operational dashboard.
- Required sales/purchase/inventory/tax reports.
- Report templates and compliance acceptance tests.

Exit criteria:
- Current official requirements are documented.
- Ambiguous submissions are reconciled without blind resubmission.
- Report totals match source records.

## Phase 6 — Payments and accounting
Deliverables:
- Payment provider integration where required.
- Receivables/payables, cash/bank reconciliation.
- If agreed: chart of accounts, balanced journal posting, GL and financial statements.
- Period close and correction workflows.

Exit criteria:
- Payment callbacks are verified and idempotent.
- Accounting journals balance and reconcile to subledgers.

## Phase 7 — Offline, analytics, notifications
Deliverables:
- Only the offline workflows approved in discovery.
- Sync status, idempotency, conflict resolution, recovery.
- Low-stock/expiry and operational notifications.
- Analytics with clearly defined formulas.

Exit criteria:
- Offline synchronization does not overwrite finalized server records.
- Conflicts are visible and recoverable.

## Phase 8 — Production readiness
Deliverables:
- Load and security tests.
- Observability, alerts, secret management.
- Backup/restore drills and RPO/RTO.
- Dependency scanning and deployment rollback plan.
- User documentation, support runbooks, client UAT.

Exit criteria:
- No unresolved critical security or financial integrity defect.
- Client signs off on acceptance tests.
- Compliance claims are limited to verified evidence.

## Recommended first vertical slice
1. Create organization.
2. Invite a user.
3. Assign a role and branch scope.
4. Create product and warehouse.
5. Receive opening stock.
6. Complete one sale.
7. Finalize invoice and stock movement atomically.
8. Verify audit events and reports.
9. Test duplicate request and cross-tenant denial.

This slice validates the foundations before broadening the product.
