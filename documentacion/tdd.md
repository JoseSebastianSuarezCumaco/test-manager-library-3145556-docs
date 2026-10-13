# TDD: tests as the evidence of each task

**Author:** José Sebastián Suárez Cumaco (GitHub: JoseSebastianSuarezCumaco)

## 1. Approach

The spec says a task is done only when its evidence exists and can be run (art. XII). `tasks.md` defines that
evidence, and in the domain the evidence is unit tests. Every domain piece is delivered together with its tests
and is not considered finished until they pass.

## 2. Rules the tests follow

- The domain does not depend on Laravel, the database or HTTP: its tests use only PHPUnit.
- Time is never read from the system: a `Clock` is injected. Tests use `Tests\Support\FixedClock`, which only
  moves when the test says so.
- Every spec rule has a test named after what it protects.

## 3. Existing tests

Command: `php artisan test --filter=<name>`

| Task | Test file | Result | What it verifies |
|---|---|---|---|
| T-03 | `tests/Unit/Domain/ValueObject/ValueObjectsTest.php` | 7 pass | 13-digit ISBN; quantity > 0; range end >= start |
| T-04 | `tests/Unit/Domain/Entity/BookTest.php` | 9 pass | Available never < 0 nor > total; total change; soft delete |
| T-05 | `tests/Unit/Domain/Entity/LoanTest.php` | 12 pass | Due in 14 days; max 3 lines; no repeated book; `open`, `overdue`, `closed`; returning twice |

Design note: `overdue` uses a strict comparison (`due_at < now`), following CA-09.1 ("before now"). One test covers
the exact due instant (still `open`) and another one second later (`overdue`).

## 4. Pending tests according to `tasks.md`

- Integration against a real PostgreSQL (T-12, T-13): two loans of the last copy, one 201 and one 409.
- HTTP contract test (T-16): every route and every error code.
- Architecture tests (T-19): they fail if the domain imports something forbidden.