# AGENT.md — Nepal Inventory, POS, Accounting & IRD Billing SaaS

## Product objective
Build a secure multi-tenant SaaS for Nepalese businesses covering inventory, multi-warehouse operations, purchasing, POS, sales, invoicing, payments, tax configuration, IRD/CBMS integration, reporting, and optional double-entry accounting.

## Technology
- Java 21+
- Spring Boot 3.x
- Spring Security
- PostgreSQL
- Maven
- Flyway
- JUnit 5 + Testcontainers
- OpenAPI
- Transactional outbox for durable integration events
- RabbitMQ only when operational scale warrants it

## Architectural rules
1. Begin with a modular monolith. Keep modules isolated behind application-service interfaces.
2. Do not share JPA entities across modules or expose them through APIs.
3. Use PostgreSQL transactions for operations that must be atomic.
4. Use `BigDecimal` for money and explicit rounding rules.
5. Enforce critical invariants in both application code and database constraints.
6. Do not claim regulatory compliance until the current applicable IRD requirements have been verified and tested.
7. Never fabricate external API success, regulatory approval, or test results.
8. Never log passwords, tokens, CBMS credentials, or unnecessary sensitive data.

## Required modules
- Platform identity and tenant lifecycle
- Organization membership, users, roles, and permissions
- Branches and warehouses
- Product catalog, variants, barcodes, and units
- Inventory ledger, stock balance projections, reservations, transfers, and valuation
- Suppliers, purchase orders, goods receipts, supplier bills, and returns
- Customers, quotations, sales orders, POS, invoices, returns, and credit notes
- Payments, refunds, receivables, and payables
- Tax engine and Nepal fiscal-calendar support
- IRD/CBMS adapter and reconciliation workflow
- Accounting and financial reconciliation, if included in agreed scope
- Statutory reports and exports
- Audit, notifications, analytics, and operations

## Identity and authorization
- Global user identity is separate from organization membership.
- A user may belong to multiple organizations, each with different roles and scope.
- Resolve tenant and organization context from authenticated server-side state, never from a client-supplied ID alone.
- Enforce authorization on every protected server operation; frontend visibility is not authorization.
- Support platform roles separately from customer organization roles.
- Organization admins cannot grant permissions beyond their authority.
- Protect the last active organization owner; require an ownership-transfer workflow.
- Membership and permission revocation must take effect promptly.
- Support organization, branch, and warehouse data scopes.
- Audit invitations, membership changes, role assignments, privileged support access, and security events.

## Financial integrity
- Draft documents may be edited under permission controls.
- Finalized invoices, issued credit notes, posted journals, finalized receipts, and committed stock movements are immutable.
- Correct finalized transactions through linked correction, reversal, or adjustment documents.
- Invoice finalization must be idempotent.
- Persist invoice, applicable stock movements, and outbox events atomically when they share a database transaction.
- Use a transactional counter with locking for invoice numbering; enforce a database uniqueness constraint.
- Never reuse finalized invoice numbers. Gap/void handling must follow verified rules.
- A network timeout does not prove that CBMS rejected a submission.
- Avoid blind retries after ambiguous external submissions; reconcile using verified API capabilities and documented procedures.

## Inventory integrity
- The stock ledger is the source of truth; balances are projections that can be reconciled/rebuilt.
- Record tenant, organization, warehouse, product, quantity, unit, movement type, source document, business date, timestamp, actor/process, and cost data as applicable.
- Corrections use linked compensating movements.
- Prevent checkout races in stock reservations and availability checks.
- Make negative-stock behavior an explicit policy.
- Model transfer dispatch and receipt separately when operationally appropriate.
- Define FIFO or weighted-average costing before valuation/COGS implementation.
- Test returns, landed costs, backdated movements, closed periods, and stock valuation.

## Nepal compliance
- Verify current IRD notices, API contract, taxpayer eligibility, electronic billing procedure, software approval/listing requirements, offline rules, and report annex definitions.
- Use effective-dated tax rules and preserve calculation inputs on finalized documents.
- Validate PAN format without claiming that format validation proves official registration.
- Use a tested BS–AD conversion library and define date/time semantics.
- Treat Annex 8, Annex 9, and Annex 13 names/layouts as requirements to verify, not assumptions.
- Version the adapter and report templates where appropriate.

## Quality gates
- Flyway migrations for every schema change.
- Unit and PostgreSQL integration tests.
- Concurrency, tenant-isolation, authorization, idempotency, and reconciliation tests.
- OpenAPI documentation and normalized error responses.
- Structured logs, correlation IDs, health checks, metrics, and alerting.
- Backup restoration drills and documented RPO/RTO.
- A feature is not complete until tests pass and documentation is updated.

## AI agent workflow
For each feature:
1. Read the relevant specification and existing code.
2. Identify dependencies and unresolved business decisions.
3. Propose schema, API contract, and acceptance criteria.
4. Implement migration and domain/application logic.
5. Add unit, integration, security, and concurrency tests as relevant.
6. Run tests and build; do not bypass failing tests.
7. Update documentation.
8. Report changed files, test evidence, risks, and unresolved requirements.

Do not generate the entire application in one uncontrolled pass. Implement and integrate one phase at a time.
