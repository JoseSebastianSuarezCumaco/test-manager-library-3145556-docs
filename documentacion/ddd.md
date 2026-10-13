# DDD: domain design of Manager Library

**Author:** José Sebastián Suárez Cumaco (GitHub: JoseSebastianSuarezCumaco)

## 1. Layers (hexagonal architecture, arts. I to III)

| Layer | Folder in `api` | Rule |
|---|---|---|
| Domain | `app/Domain` | No dependency on Laravel, the database or HTTP |
| Application | `app/Application` (pending) | Use cases with their own ports |
| Adapters | `app/Adapters` (pending) | Persistence, security, covers and HTTP, outside the domain |
| Composition | A single service provider (pending) | The only place where ports and adapters are wired |

## 2. Business language and code names

| Business term | Code |
|---|---|
| Book, available copies, soft delete | `Book`, `available`, `softDelete` |
| Loan and loan line | `Loan`, `LoanItem` |
| Take out and put back a copy | `withdraw`, `restock` |
| Return a line | `returnItem` |
| Calculated state (open, overdue, closed) | `LoanStatus::Open`, `Overdue`, `Closed` |

## 3. Value objects

`Isbn` (13 digits), `Quantity` (greater than zero), `DateRange` (end >= start), `LoanStatus` (enum).
They are immutable and validated on creation: an invalid value cannot exist.

## 4. Entities and aggregate

- **`Book`** protects RN-1: available copies are always between 0 and the total. `withdraw` fails when there are no
  copies; `restock` fails when all copies are already there; `changeTotal` moves available by the same difference and
  never leaves it below zero (CA-03.3). Soft delete only sets `deletedAt`.
- **`Loan`** is the aggregate root and holds its `LoanItem`s. It enforces: at most 3 lines, no repeated book,
  due date 14 days after `loaned_at`, and return per line.
- **`LoanItem`** freezes the title, ISBN and category name at loan time (RN-4) and can be returned only once (CA-07.2).
- **The loan status is calculated** with a `Clock` and never stored (CA-08.3).

## 5. Where each rule lives

| Rule | Where |
|---|---|
| Available between 0 and total | `Book` in the domain, and the `ck_book_available_range` CHECK in the database |
| 14 days, 3 lines, no repeats | `Loan` |
| Max 3 unreturned per member and member with overdue | Lend use case (T-09, pending): needs member data |
| A book with open loans cannot be soft-deleted | Books use case (T-08, pending): `Book` does not know about loans |
| Last copy with two librarians at once | Persistence adapter (T-13, pending): atomic `UPDATE` |

## 6. Domain errors

All violations extend `DomainRuleViolation`. The HTTP adapter (T-16, pending) will translate them to the codes of
`api-contract.md` inside the single error body.

## 7. Domain pending

`Member` and `User` (T-06): username normalization and valid role.