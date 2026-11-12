# Jira And Confluence As Requirements

Use when a task starts from a Jira key (`PIN-10013`), a Confluence page ID or URL, or an SRS/SERS. Fetch them with the available Atlassian tools. They are read-only unless the user asks for a write.

## Jira: What To Extract

- Key, type, summary, status, assignee, Fix Version.
- Description, acceptance criteria, test notes.
- Parent, linked issues, blockers.
- Recent comments with decisions or changed scope.
- Confluence links and page IDs. Do not claim an SRS exists unless a field, comment, link, or the user confirms it.

If only keywords are given, search and show the candidates before treating one as authoritative.

## SRS/SERS: How To Read It

An SRS is a proposal to validate. It mixes three layers:

- **Domain meaning of fields and rules:** honor it.
- **Public contracts (paths, payloads, status codes):** copy them exactly.
- **Illustrative snippets:** translate them into the project's conventions. Never copy `snake_case` or pseudocode.

Check every SRS field name against existing domain models before committing. SRS pages can carry typos and stale names.

A layer marked "future work" is out of scope: add no enum cases, converter branches, or handlers for it. Restrict local types to the supported subset so exhaustive matching stays valid.

Also extract actors, migrations, feature flags, errors, permissions, notifications, out-of-scope items, and open questions.

## Handoff Before Coding

```text
Jira: PIN-10013 - <summary> (<status>)
Confluence: <page title> (<page id/version>)
Scope: <what must change>
Packages: <names and why>
Source of truth: <OpenAPI/protobuf/SQL/domain/config/service code>
Tests: <files or commands>
Open questions: <only blockers>
```

The ticket sets the outcome. The repository sets the conventions.
