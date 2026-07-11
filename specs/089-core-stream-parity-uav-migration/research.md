# Decisions

- Continuous publication needs stream session/sequence state; finite objects do
  not and remain segmented exact-name Data.
- FEC metadata may be carried generically, but repair codec/policy belongs to
  the application.
- Python parity is achieved by binding the C++ engine, not maintaining tests
  against two implementations indefinitely.
