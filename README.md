# milestone-runner

The trusted test runner for [Milestone](https://github.com/TanmayShah29/milestone-ai-escrow), pay-on-proof
payments on PayPal. Milestone counts a result only when this repository's `.github/workflows/run.yml` on
`main` signed it (evidence level E3). `main` is protected: no force pushes, no deletion.

**How a run works**, started by Milestone with `repo, pr, head_sha, fingerprint, nonce`:
1. Fetch the signed contract by its fingerprint and check its SHA-256. Its repository must be `repo`.
2. Check out the PR at `head_sha` as data. Nothing from it runs on the host.
3. For each tested criterion, use a fresh pair of containers:
   - **worker:** the PR's code behind `harness/shim.mjs`, with no network, a read-only root, no
     capabilities and uid 1000;
   - **tests:** the signed test, with each import of the worker's code replaced by a proxy
     (`harness/plan.mjs`). The proxy calls it synchronously over a Unix socket (`harness/client.mjs`).
   The worker's code can only change the values it returns. It can't touch the assertions, the reporter
   or the results.
4. Write `results.json` (nonce, repo, PR, commit, fingerprint, image digest, one result per criterion).
   Sign it with `actions/attest` (GitHub OIDC + Sigstore, public Rekor log) and upload it.

Milestone then checks the signature and this workflow's exact identity, the digest of `results.json`, the
predicate, and the nonce, commit and contract (`src/lib/review/runner.ts` in the app). Anyone can check a
result themselves:

```
gh attestation verify results.json -R TanmayShah29/milestone-runner \
  --predicate-type https://milestone-ai-escrow.onrender.com/attestations/test-results/v2 \
  --signer-workflow TanmayShah29/milestone-runner/.github/workflows/run.yml
```

**Limits:** values cross the boundary as JSON, plus `undefined`, `NaN`, `Infinity`, `Date` and thrown
errors. Tests run on Node 24 (JavaScript and TypeScript with type stripping), not Python yet.

The Phase 0 spike (fixtures `honest`, `evil`, `exiter`) is in this repository's history at `be17b02`.

MIT licensed.
