# Research

- **Decision**: run 02 is `ground-telemetry-not-visible`: drone reports armed
  0.349 s after Arm response, but Ground Station never observes armed before expiry.
- **Decision**: run 05 is `final-observation-missed`: Ground Station logs armed
  about 70 ms before expiry, inside the 100 ms polling gap.
- **Decision**: perform a final cached read, not an extra request or longer wait.
- **Rejected**: retry, timeout extension, faster polling, or packet-direction inference.
