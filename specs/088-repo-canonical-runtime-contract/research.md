# Repo Runtime ADR

**Decision**: C++ object/local-protocol authority with one Python NDNSF network
operations adapter.

## Evaluation

| Criterion | C++ contract | DI-local Python implementation |
|---|---|---|
| Exact Data wire validation | Native `ndn::Data` parse and immutable conflict check | Duplicates policy around native producer |
| Public object API | `RepoClient` and pybind adapter | Large mixed DI/Repo module |
| SQLite + bounded cache | Implemented and restart-tested | Additional catalog/repair SQLite state |
| HA/repair orchestration | Needs Python operational adapter | Implemented but misplaced |
| Security | Typed local contract; duplicate unversioned network entry removed | Uses Core V2/Targeted with versioned public/private operation policy |
| Maintainability | Small typed contract | `repo.py` combines many responsibilities |

The alternatives are complementary only at the boundary: Python orchestrates
the canonical C++ object contract but may not redefine object semantics. It is
the only deployed network runtime. A timing-only comparison is insufficient and
did not drive this decision.
