# saif-store-platform

Full-stack apparel e-commerce storefront and admin console built with React, TypeScript, Vite, Tailwind CSS, and Supabase (PostgreSQL, Auth, RLS, Storage), verified with an offline PGlite PostgreSQL test suite.

## Engineering Highlights

- **Atomic checkout RPC (`place_order`):** Validates line items against live inventory using row-level locks, recomputes subtotals, discounts, and shipping server-side, consumes coupons, reserves stock, and writes the order, order items, payment record, and timeline event inside a single PostgreSQL transaction.
- **Manual payment verification state machine:** Supports Egyptian InstaPay and Vodafone Cash transfers. Customers upload payment receipts to a private Supabase Storage bucket; `submit_payment()` moves the record to `under_review`, and admins review receipts via `review_payment()` to approve, reject with a required reason (allowing resubmission), or cancel (automatically restoring reserved stock). Payments are never auto-approved.
- **Database-enforced security boundaries:** Role privilege escalation is blocked at the PostgreSQL layer via column grants, triggers, and Row-Level Security. Payment proof files are readable only by their owner and verified admins, and coupon codes are never exposed to unauthenticated clients.
- **In-process PostgreSQL integration tests (PGlite):** The Vitest suite (10 test files, 138 tests) executes the real `supabase/` SQL schema, RLS policies, functions, and upgrade migrations against `@electric-sql/pglite` (WebAssembly PostgreSQL) with mocked `auth`/`storage` schemas — verifying checkout atomicity, stock reservation/restoration, RLS tenant isolation, role-escalation defenses, and idempotent schema upgrades without external network dependencies.

## Stack

- **Frontend:** React 18, TypeScript, Vite, React Router 6, Tailwind CSS
- **Backend / Data:** Supabase (PostgreSQL, Auth, Row-Level Security, Storage)
- **Testing:** Vitest 4, React Testing Library, jsdom, `@electric-sql/pglite`

## Project Structure

```text
src/
├── pages/        # Storefront routes and /admin console views
├── components/   # Storefront UI, cart drawer, product gallery, admin widgets
├── context/      # Auth, cart, settings, and storefront state
├── hooks/        # Data fetching, pagination, filters, and admin mutations
├── i18n/         # Bilingual English / Arabic translations and RTL handling
└── lib/          # Supabase client, pricing helpers, validation, constants
supabase/
├── schema.sql    # Core tables, constraints, and indexes
├── functions.sql # Transactional RPCs (place_order, submit_payment, review_payment)
├── rls.sql       # Row-Level Security policies and storage access rules
├── setup.sql     # Consolidated initial setup script
├── migrations/   # Incremental schema and CMS upgrades
└── diagnostics/  # Schema and admin verification queries
tests/            # Unit, jsdom render/route sweep, and PGlite DB integration suites
```

## Local Setup

1. Install dependencies:

```bash
npm install
```

2. Configure environment variables:

```bash
cp .env.example .env
# Set VITE_SUPABASE_URL and VITE_SUPABASE_PUBLISHABLE_KEY
```

3. Apply `supabase/setup.sql` in a new Supabase project's SQL Editor (or run the incremental files under `supabase/migrations/` for existing databases).

4. Create an admin user by registering normally, then promoting the profile in SQL:

```sql
UPDATE profiles SET role = 'admin' WHERE email = 'you@example.com';
```

5. Run locally or execute verification checks:

```bash
npm run dev
npm run lint     # TypeScript type-check (tsc --noEmit)
npm test         # Unit, jsdom smoke, and PGlite PostgreSQL integration tests
npm run build    # Type-check + production Vite bundle
```

## License

[MIT](./LICENSE)
