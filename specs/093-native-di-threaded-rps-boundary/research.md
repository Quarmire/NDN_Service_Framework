# Research Notes

Spec 092 validates threaded mode at 1 RPS over three matched runs. The existing
`NDNSF_DI_RuntimeAware_RpsSweep.py` is not used because it hard-codes the
deterministic runner and admission lease and classifies stability only by
success rate. Those controls and gate semantics do not match this experiment.

This feature uses direct canonical harness commands and predeclared scheduling
plus system stability gates.
