# milestone-runner

Trusted test runner for [Milestone](https://github.com/TanmayShah29/milestone-ai-escrow), pay-on-proof payments
on PayPal. This is a Phase 0 spike.

The worker's code and the signed acceptance tests run in **two separate containers with no network**,
talking over a Unix socket. The worker's code can only change the values it returns. It can't touch the
assertions, the test reporter or the results file. `results.json` is signed with a GitHub artifact
attestation (Sigstore), and Milestone checks the signature and this workflow's identity before it counts
the results.

Fixtures:
- `honest`: correct code;
- `evil`: patches `assert`, tries to fake the report, rewrite the harness and use the network;
- `exiter`: calls `process.exit(0)` mid-test.

`run-local.sh <fixture>` runs the same containers locally.

MIT licensed.
