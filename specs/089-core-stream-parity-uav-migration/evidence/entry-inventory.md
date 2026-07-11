# Entry Inventory

- Core C++ already defines stream value objects, producer/reorder buffers,
  metrics, health and adaptive fetch state with unit tests.
- `pythonWrapper/ndnsf/streaming.py` duplicates producer/reorder/adaptive logic.
- `_ndnsf` does not currently expose the C++ state engine.
- UAV maps video packets/FEC metadata and adaptive state to Core types, while
  its ground-station loop still owns video-domain behavior.
