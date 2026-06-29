# Tasks: DI Auto Runtime Assignment

- [x] A001 Create Feature 018 spec, plan, and task list.
- [x] A002 Add `--assignment auto` resolution in MiniNDN harness.
- [x] A003 Preserve fixed-assignment behavior and summary fields.
- [x] A004 Validate Python syntax.
- [x] A005 Run local auto smoke for concurrency 1, 2, and 4.
- [x] A006 Run minimal full-network auto smoke for concurrency 1, 2, and 4.
- [x] A007 Record accepted results.

## Result

Accepted auto smoke:

- `/tmp/ndnsf-di-auto-c1/summary.json`: `auto -> single-provider`,
  `selectedCandidate=single-provider-serial`.
- `/tmp/ndnsf-di-auto-c2/summary.json`: `auto -> default`,
  `selectedCandidate=shared-backbone-current`.
- `/tmp/ndnsf-di-auto-c4/summary.json`: `auto -> default`,
  `selectedCandidate=shared-backbone-current`.

Fixed assignment smoke:

- `/tmp/ndnsf-di-fixed-default-smoke/summary.json`: fixed `default` passed.
- `/tmp/ndnsf-di-fixed-single-smoke/summary.json`: fixed `single-provider`
  passed.

Full-network auto smoke:

- `/tmp/ndnsf-di-full-auto-c1/summary.json`: `auto -> single-provider`,
  `userExecution=executed`, `dependencyExecution=executed`.
- `/tmp/ndnsf-di-full-auto-c2/summary.json`: `auto -> default`,
  `userExecution=executed`, `dependencyExecution=executed`.
- `/tmp/ndnsf-di-full-auto-c4/summary.json`: `auto -> default`,
  `userExecution=executed`, `dependencyExecution=executed`.
