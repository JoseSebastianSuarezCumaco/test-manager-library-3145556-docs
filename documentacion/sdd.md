# SDD: how Spec-Driven Development was applied

**Author:** José Sebastián Suárez Cumaco (GitHub: JoseSebastianSuarezCumaco)

## 1. How the spec was read

The reading order of the `docs` README was followed: `constitution.md`, `spec.md`, `plan.md`, `architecture.md`,
`arquitectura-panoramica.md`, `data-model.md`, `api-contract.md`, `tasks.md` and `adr/`. The `spec-python/` folder
was used; `spec.md`, `data-model.md` and `api-contract.md` were checked and are identical in `spec-.net/`.

## 2. Stack translation

The README asks to respect the **business** (user stories, acceptance criteria, rules, API contract, data model)
and to translate the **technology**.

| In the spec (Python/.NET) | In the challenge (Laravel/React) |
|---|---|
| Application migrations | Laravel migration with raw SQL (`database/migrations`) |
| Hexagonal layers | `app/Domain`, `app/Application`, `app/Adapters` and a single composition service provider |
| `import-linter` (T-19) | Equivalent architecture tests (pending) |
| Tests with a fake `Clock` | `App\Domain\Clock` interface and `Tests\Support\FixedClock` |

## 3. Document hierarchy and detected "traps"

When two documents contradict each other, `data-model.md` wins (art. X). Decisions taken:

- **ADR-002 mentions a "version witness"** column in `book`, but `data-model.md` has no such column. Decision:
  optimistic concurrency without a new column, using an atomic `UPDATE`
  `SET available_copies = available_copies - 1 WHERE id = ? AND available_copies >= 1` inside the transaction.
  If it affects 0 rows, the answer is 409 `BOOK_NOT_AVAILABLE`. The `ck_book_available_range` CHECK is the last barrier.
- **Data model untouched:** no extra columns and no `DEFAULT`. All DDL lives in the application migrations
  (ADR-001); `infra` has no `CREATE TABLE`.
- **`ck_book_total_positive` and `ck_user_role`** are marked "pending (T-18)": they are not in the initial migration.
- **The admin is neither seeded nor created through the API:** it is created at startup from `ADMIN_USERNAME` and
  `ADMIN_PASSWORD` (T-17).
- **In Laravel, `GET /loans/overdue` is declared before `/loans/{id}`.**
- **A single error body** `{code, message, details}`, never stack traces.
- **Language:** code, commits, branches and documentation are in English. User-facing `message` fields of the API
  errors are in Spanish because the spec asks for it (RN-6, art. XI); this is pending confirmation.
- No branch or commit conventions were invented: the spec does not impose any. Small commits (one per task) with
  progress pushed were made, as the README asks.

## 4. Evidence per task (art. XII)

| Task | Status | Evidence |
|---|---|---|
| T-01 Skeleton and `GET /health` | Done | `curl http://localhost:8000/api/v1/health` returns `{"status":"ok"}` |
| T-02 Initial migration | Done | 5 seeded categories and 0 columns with a `DEFAULT` (`psql` queries) |
| T-03 Value objects | Done | 7 unit tests pass |
| T-04 `Book` | Done | 9 unit tests pass |
| T-05 `Loan` | Done | 12 unit tests pass |
| T-06 onwards | Pending | The reason is documented in the `api` README |