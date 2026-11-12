# Jira Tickets (PIN)

The Manuale sets the issue type, Fix Version, and links. The Writing Rule in `SKILL.md` sets the text.

## Rules

- Jira is read-only unless the user asks for a write. Drafting text does not authorize creating or editing an issue.
- Write in Italian. Keep endpoints, fields, and error messages verbatim.
- The reader is a teammate who knows the project. Do not explain e-service, tenant, or readmodel.
- Name the endpoint, field, or visible behavior. Do not write "la propagazione", "il supporto", "la catena".
- No file paths, function names, code analysis, or tables. Those go in the PR.
- One ticket, one outcome. Split unrelated work.
- Do not invent requirements or SRS links. Fetch the current ticket before rewriting it.

## Issue Type

| Commit type | Jira issue type |
| --- | --- |
| `feature` | `[Interop] Epic`, `Story`, `Backend`, `Frontend` |
| `fix` | `[Interop] Bug` |
| `hotfix` | `[Interop] Hotfix` |
| `refactor`, `test` | `[Interop] Story`, `Backend`, `Sub-task Backend`, `Frontend`, `Sub-task Frontend` |
| `perf`, `chore` | As `refactor`, plus `Infra - Delivery`, `Sub-task Infra` |
| `docs` | `[Interop] Design - Delivery`, `Sub-task Design` |
| `build`, `ci` | `[Interop] Infra - Delivery`, `Sub-task Infra` |
| `align` | `[Interop] Backend`, `Sub-task Backend`, `Frontend`, `Sub-task Frontend` |

- `Story`: only for work with sub-tasks of different kinds.
- `Backend` or `Frontend`: only for work in that one domain, such as a refactor.
- `Bug`: only for observable wrong behavior, never for technical debt.

## Summary

One line that says what must be done, readable without opening the ticket. Do not use "sistemare", "migliorare", or "investigare" without a concrete result.

## Description

Write the shortest form that fits the ticket.

**Sub-task or reminder:** 2 or 3 sentences, no headings.

**Bug:**

```text
### Passi per riprodurre
1. <Azione con ruolo, entità e valori concreti.>

### Atteso
<Risultato esatto.>

### Osservato
<Risultato esatto, con status code o messaggio.>
```

**Task or story for someone else:**

```text
### Problema
<Cosa non va oggi, in una o due frasi.>

### Cosa fare
<Risultato richiesto.>

### Criteri di accettazione
- <Condizione verificabile.>
```

Add `### Fuori scope` only for a plausible adjacent change you exclude. Add `### Riferimenti` only for confirmed SRS pages or related tickets.

A rewrite for clarity must come out shorter.

## Acceptance Criteria

Each one can be verified on its own, with role, state, and result. Do not write "funziona correttamente", "il codice è più pulito", or "tutti i test passano".

A behavior-preserving refactor says which behavior must not change.

## Metadata

- Link dependencies with Jira relations (`blocks`, `is blocked by`, `relates to`), and link the parent or Epic.
- Set Fix Version before merge. Do not guess it while drafting.

## Reviewing A Ticket

Answer **Ready** or **Not ready**. Not ready means a missing outcome, reproduction, acceptance criterion, dependency, or decision changes the implementation. Propose exact replacement text for each blocker.
