# Quickstart

```bash
./build/unit-tests --run_test=Stream*
PYTHONPATH=pythonWrapper python3 -m unittest discover \
  -s tests/python -p 'test_ndnsf_core_streaming.py' -v
```
