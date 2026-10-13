# SPECIFICATION.md — Nepal Inventory & Billing SaaS

## 1. Product assumptions
- Multi-tenant SaaS from the start.
- One tenant may manage one or more legal/business organizations.
- An organization may have multiple branches and warehouses.
- One user identity may belong to multiple organizations.
- Initial architecture is a modular monolith.
- IRD/CBMS compliance requirements are verified with the client and qualified Nepal tax/compliance advice before production claims.

## 2. Functional modules

### 2.1 Platform administration
Tenant lifecycle, service status, subscription plan hooks, support tooling, platform audit, and operational monitoring. Platform support access to customer data must be limited, purpose-bound, and audited.

### 2.2 Users and access control
User invitations, profiles, activation/suspension, password reset, optional MFA, session revocation, organization membership, branch/warehouse scope, custom roles, permission catalog, access review, security events, and audit logs.

Starter roles:
- Platform Super Admin
- Platform Support Agent
- Organization Owner
- Organization Admin
- Branch Manager
- Salesperson/Cashier
- Inventory Manager
- Purchasing Manager
- Accountant
- Auditor/Viewer

Permissions should be granular, e.g. `sales.invoice.finalize`, `inventory.stock.adjust`, `payments.refund`, `reports.export`, `roles.assign`.

### 2.3 Catalog
Products, services, SKUs, barcode, variants, categories, brands, units/conversions, purchase/sales prices, price lists, discounts, tax classifications, imports/exports, archival.

### 2.4 Inventory
Warehouses, opening balances, receipts, sales dispatch, customer returns, supplier returns, transfers, stock adjustments, damaged/expired stock, counts, reservations, batch/lot, serial number, expiry tracking as needed, ledger, balance projection, valuation and reconciliation.

### 2.5 Purchasing
Suppliers, purchase orders, partial receipts, goods received notes, supplier bills, purchase returns, supplier payments, landed costs, supplier statements, purchase registers.

### 2.6 Sales and POS
Customers, quotations, sales orders, POS checkout, barcode workflow, discounts, payment methods, credit sales, partial payments, receipts, returns, refunds, customer statements, shift/cash-drawer reconciliation when needed.

### 2.7 Billing and tax
Draft/final invoices, immutable finalized documents, sequential fiscal-year numbering, effective-dated tax rules, tax snapshots, credit notes and correction workflow, reprint audit/watermark, invoice PDF, customer PAN fields, audit trail.

### 2.8 IRD/CBMS
Dedicated adapter, documented authentication, schema mapping, submission states, durable outbox/queue, retries with backoff, ambiguous-result reconciliation, sanitized attempt logs, operational dashboard, versioned fixtures. Confirm current official requirements before implementation.

### 2.9 Payments and receivables/payables
Payment records, invoice allocation, partial settlement, refunds/reversals, provider callback verification, customer receivables, supplier payables, cash/bank reconciliation.

### 2.10 Accounting (scope decision)
If full accounting is required: chart of accounts, balanced journals, general ledger, trial balance, P&L, balance sheet, cash flow, period close, posting rules, reversals, and reconciliation. A qualified accountant must approve accounting mappings and reports.

### 2.11 Reports
Sales/purchase books, inventory movement, stock valuation, tax summaries, receivables/payables, customer/supplier statements, audit reports, PDF/Excel exports, report versioning and export audit. Verify required annex numbering/layout with current IRD guidance.

### 2.12 Offline operation
Optional local queue and sync for agreed workflows only. Define what can be created offline, legal issuance rules, collision resolution, idempotency, stock conflicts, numbering, recovery, and user-visible sync states.

### 2.13 Analytics and notifications
Low stock, expiry, sales summaries, margin, purchasing trends, pending CBMS submission, rejection and integration failure alerts. Distinguish estimates from reconciled financial values.

## 3. Core domain rules
- Financial amounts use `BigDecimal`.
- Drafts are editable under permissions; finalized financial documents and posted stock movements are immutable.
- Correction documents are linked to the original.
- Every write operation is tenant-scoped and authorized.
- Invoice finalization is idempotent and uses a concurrency-safe transactional counter.
- Stock movements are ledger entries; balances are reconciled projections.
- Invoice, stock movements, and outbox event commit atomically when in the same database.
- CBMS retries are durable and must account for ambiguous responses.
- Tax rules are effective-dated; historical tax snapshots are retained.
- BS/AD date conversion uses a tested library and explicit semantics.
- Negative stock, costing method, returns treatment, fiscal year, and approval thresholds are configured business decisions.

## 4. Non-functional requirements
- Tenant isolation and least privilege.
- Auditability and traceability.
- Transactional integrity and idempotency.
- Secure secret handling.
- API documentation and versioning.
- Pagination and query limits.
- Observability, backups, restore tests, incident handling.
- Accessibility and usable POS workflows.
- Performance targets to be agreed based on expected scale.
- Data retention and privacy policy to be agreed before launch.

## 5. Decisions requiring client confirmation
1. Business types: retail, wholesale, distribution, manufacturing, or mixed.
2. Tenant/org/branch structure.
3. Whether full double-entry accounting is in initial scope.
4. Offline POS scope and legally permitted workflow.
5. Costing method: FIFO or weighted average.
6. Negative stock policy.
7. Approval thresholds for discounts, refunds, stock adjustments, and credit notes.
8. Payment providers and supported payment modes.
9. Required reports and verified IRD annex definitions.
10. Current CBMS/API onboarding and approval requirements.
11. User MFA and password/session policies.
12. Expected tenants, users, invoices/day, products, and warehouses.
13. Subscription plans, quotas, billing, and tenant suspension rules.
14. Data retention, backup, recovery objectives, and hosting region.

## 6. Acceptance criteria
- Cross-tenant reads/writes are blocked and tested.
- Role and scope restrictions are enforced server-side.
- Concurrent invoice finalization cannot duplicate numbers.
- Duplicate requests do not duplicate invoices, payments, or stock movements.
- Failed transactions leave no partial local financial state.
- Stock balances reconcile to ledger movements.
- Tax calculations match approved fixtures.
- CBMS timeouts and retries do not cause blind duplicate submissions.
- Reports reconcile to source records.
- Backup restoration is demonstrated.
- Client UAT and compliance checklist are signed off before production.
