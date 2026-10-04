# Test requirements

## Commands & reproducibility
1. Every command (in docs and chat) must be written in full, with all arguments explicit.
2. All CLI arguments are required — no implicit defaults that silently change behaviour.
3. Commands must fail fast, stating exactly which argument to set, when one is missing.
4. The goal is that a random person on a random network/device can run the tests from the written commands alone.
5. The test commands are run manually by the human, not by an AI that can resolve intermediate issues.
6. An argument must be required only if silently defaulting it could make the run test the wrong thing or hit the wrong target unnoticed; values that are purely derived from a required argument (e.g. EXTRA from FLAKE) or are mere timeouts may keep defaults.

## Test matrix & scope
7. Provide integration tests covering the whole UI / Flutter app.
8. Every test has an accompanying recording, and the recording must correspond to what the test actually tests.
9. Determine which tests can run on a single deployment and which interfere (require a fresh deployment).
10. A test requires a fresh deployment when it changes settings/state that another test relies on relative to the clean/initial deployment.
11. Group tests by whether they can share one deployment vs. must be isolated.
12. Each test declares the repos it depends on; this scopes both its state identity and its clean-state gate.
13. Structure: test group (A) with a command to run the whole group; tests within a group (A.1) each with a command to run just that test.
14. Keep `N/A` for entries/combinations that are not available.

## Networks
15. Supported networks: Tor, Tor+https, Chutney, Chutney+https, Https, Yggdrasil, Hyphanet.
16. Per test, record which network SelfPrivacy uses.
17. Per network, record in which network setup the test was run.
18. Per network setup, record whether it was run Locally or on CI.

## Network setups
19. `vm-local`: VirtualBox VM on the host (build-and-run.sh), no external wiring.
20. `lan-setup-0a`: direct cable, router out of path, netboot install; cable later replugged into the router for its IP; no wifi config.
21. `lan-setup-0b`: as 0a, plus a wifi config for the same router R.
22. `lan-setup-0c`: as 0a, plus a wifi config for a different router B.
23. `lan-setup-0d`: direct-cable install; cable later removed; target runs wifi-only (wifi reaching the internet).
24. `lan-setup-1`: laptop on router R's wifi, target wired to R (requires R configured for netboot).
25. `lan-setup-2`: laptop and target both on router R's wifi (requires R configured for netboot).
26. `usb-0a`: installer USB → target internal disk (manual boot); target wired to router R for internet.
27. `usb-0b`: as usb-0a, plus a wifi config for the same router R.
28. `usb-0c`: as usb-0a, plus a wifi config for a different router B.
29. Use one diagram per setup family (direct-cable family, usb family) to avoid duplication, without altering any option.

## Stamping & deployment identity
30. Tests must only ever run against the most recent / desired code state.
31. The system must be able to determine whether a deployed box corresponds to a given code state.
32. The target is stamped at deployment with the identity of what was deployed.
33. The stamp is based (hash-wise) on the list of repos, branches and commits used for the deployment.
34. Dirty deployments are not permitted; if dirty, that must be indicated with a hashsum of the dirtiness.
35. A test that uses the target retrieves the stamp from the device and links it to the code states it knows (locally or in committed history).
36. A test that uses the target verifies the stamp first and refuses to run if it is not the desired state.
37. This applies to every install method, including USB.
38. A manual/USB test is one uninterrupted chain: prompt to insert the USB and confirm "did you install it on the target?", then continue into the test.
39. When the box cannot be verified, distinguish "reachable but not stamped" from "unreachable".
40. Tests error if the flake.lock does not correspond to the selfprivacy-api checkout (rev differs or checkout dirty).
41. The dashboard shows an additional warning when the api checkout ≠ the deployed flake.lock pin (similar to the dirty warning).

## Dirty / clean state & integrity
42. At the start of a test, verify the code state is not dirty; if code changed, ask to commit the current state before running, so all tests run on the same code.
43. The clean-state gate checks all repos a test uses, not just one.
44. A submodule's working checkout must match the superproject's recorded pointer (no drift); otherwise refuse.
45. Test-result data must not be a factor in the clean-state check (so multiple tests can run back-to-back without each run dirtying/altering the code state).

## Results storage
46. Commit all test-result data except videos.
47. Videos are local-only artifacts; never committed.
48. Store results in a separate repo (testresults) so recording results never dirties or changes the code commit.
49. Results are keyed by the code state, so recording a result does not change that state.
50. Easily see which commands were already run and what their output was, per commit/state.
51. Be able to correct what is recorded for which test.

## Dashboard / visualization
52. An additional dashboard, based on the current one.
53. Scroll through branches and commits (states) and reload the accompanying test history on selection.
54. Hierarchy per test: test group → test → network → network setup → Local/CI.
55. The selectable state at the branch/commit level is the whole repo combination (all repos/commits used), not a single repo.
56. Per test run configuration, fold down the list of past runs for that configuration on the same state.
57. Collapsing a test shows only the latest result for each configuration.
58. Show pass / fail / N/A.
59. Click a status dot to see that run's log/output.
60. For local runs, view the recordings (both screenshots and videos).
61. Commands are not always shown; they appear on hover, with a clickable copy item in the top-right corner.
62. Hover/click the dirty-state warning to see which repos and branches are dirty.
63. Show all repos and branches a test uses (even when clean) and the commits per branch per repo, so the exact config under test is visible.
64. Display the deployed flake.lock pins for the selected state.

## CI vs local
65. Record, per run, whether it was executed Locally or on CI.
66. In CI, the recorded repo/branch/commit is whatever CI checked out (the committed refs from the git server).

## Dashboard / visualization (status model)
67. The matrix is catalog-driven: it shows every test as the overview even with no results — each applicable network × setup × {Local,CI} cell defaults to ☐ todo.
68. Cell statuses are todo / pass / fail / N/A (N/A for an unsupported network or a non-applicable setup).
69. Hovering a ☐ todo cell shows the exact command to run that test (with a copy button).

## Setup availability (preflight)
70. At the start of a test, verify the specified setup is available, as far as it can be checked.
71. USB setups: a USB device is present (and empty, if emptiness is required).
72. Wifi setups: a wifi spec (SSID, plus credentials as needed) must be specified.
73. Verify the specified wifi is in range / yields internet only when that is possible without disturbing the current wifi connection (e.g. a scan confirms the SSID is in range; internet-through-it is not checked if that would require connecting).

## Dashboard / visualization (explanatory hovers)
74. Hovering a test group (level) or a test shows a brief explanation of what it tests (sourced from the catalog `levels`/`desc`).
75. Hovering a ⚪ N/A cell shows a brief reason it is not available: a roadmap transport that is not implemented yet (catalog `future_networks`), or not applicable because that test does not exercise that transport.
76. The matrix shows a sub-header row under the setup columns indicating the Local/CI split (each cell holds two dots: local, then CI).

## Integrity — a result must reflect the committed code (no silent green)
77. A pass must mean the committed code of the recorded state actually ran; any divergence between what runs and the intended code must be detected — refuse or fail, never a silent pass.
78. Pins recorded for a run, and the pin gate, are read from the flake.lock that actually governs that run's build: a self-contained test uses its OWN flake.lock, not the backend's.
79. The pin gate checks every input the governing lock pins (e.g. selfprivacy-api AND manager), not only the api.
80. A nix build that reused an existing output (nothing built; no VM ran) is flagged (from_cache), not shown as a fresh run.
81. An output fetched from a binary cache (substituted — built elsewhere, not run here) is flagged.
82. A self-contained test's nixpkgs matches the deployed backend's nixpkgs (test ↔ production parity).
83. L3 drives the app with a pinned toolchain (the app flake's flutter) and resolves deps with the lockfile enforced; a toolchain/dep mismatch fails loud instead of silently regenerating the lockfile.
84. The box-identity check measures the box's actual running system (/run/current-system vs a baseline recorded at install), not only the install-time stamp.
85. A run forced past a gate (--allow-dirty) is recorded and shown as forced — never indistinguishable from a clean pass.
86. Manually-reported (click-through/video) results are marked manual/self-attested, distinct from automated runs.
87. A transient/flaky retry attempt is retained (marked flaky), not deleted, so intermittent failures are not hidden.
88. A checkout behind its tracked upstream blocks the run (after refreshing the remote ref); bypass only with --allow-dirty (then recorded as forced).
89. A live target address (e.g. the backend .onion) is read live from the backend, not from stale local state.

## Integrity — to implement (Track 2)
90. L3 real-data assertions run against a known-clean backend baseline: mutable state (userdata, enabled services, users, DB) is snapshot/restored (or reset) per flow so a result reflects the code, not leftover state; what the reset cannot cover is logged.
91. The clean-state check also catches build-affecting inputs invisible to `git status`: gitignored files (e.g. users.nix, .env) and a non-hermetic nix config (sandbox disabled / --impure).
92. The recorded state identity is the exact build closure (derivation path/hash), not only the three input pins.

## Dashboard / visualization (run history & flakiness)
93. One circle never represents multiple runs; each run is its own circle.
94. A config's runs are shown chronologically: collapsed shows only the most recent; expanded shows the full sequence, newest on top → oldest on the bottom.
95. Flakiness is represented by the green/red sequence of per-run circles, not by an aggregate marker or count.
96. Each run circle links to that run's log/media.
97. A run carrying trust qualifiers (forced/manual/cached/substituted/behind) is marked per-run (e.g. a ring), distinct from a clean run.

## Integrity — L3 backend targeting (Track 2, item 1a)
98. L3 desktop flows run against the backend named by the test args (`--on <setup>`): `vm-local` → the local VirtualBox VM; `lan-setup-0a..0d` / `lan-setup-1` / `lan-setup-2` / `usb-0a..0c` → a deployed physical box reached over the LAN. The backend is never hardcoded to the VM.
99. Running an L3 flow against a box requires `--ip`, `--key`, and `--token`; the box's API token is per-box and secret, so it is never defaulted (reproducibility reqs 1–6). Missing args fail fast, naming what to set.
100. For a box the dialed address is read LIVE from the box (req 89), never from stale local state: the onion from `/var/lib/tor/hidden_service/hostname`, the https host as `api.<domain>` where `domain` comes from the box's `/etc/nixos/userdata.json`.
101. `chutney` is a laptop-local private Tor network, valid for `vm-local` only; a box is reached over `tor` (real onion) or `https` (public domain, system-trust LE cert — no tunnel/mkcert CA). An unsupported network×setup combination is rejected fast.
102. An L3 run records the real setup as its `method` so the result lands in the correct matrix column; for a box, the deployment provenance the box-identity gate computed (stamp, measured running-system, box repos+pins) is recorded on the run.
103. The clean-state, pin, and behind-upstream gates apply to every gated run — a test (identified by its `level`) as well as an install — not only installs. The L3 inner recorder does not re-gate a run whose outer run_entry already gated it.

## Integrity — clean per-L3 backend-state baseline (Track 2, item 1b, req 90)
104. Around each L3 flow the backend's mutable SP state (`/etc/nixos/userdata.json`: enabled services = `modules.*.enable`, users, dns/server/provider config) is snapshotted before the flow and restored after, so a real-data assertion reflects the code, not leftover state. Target-aware (the VM or the box the run used).
105. The pre-flow state is compared to a captured known-clean golden baseline (keyed by the deployed backend's state identity + setup); drift is recorded (`l3_baseline_match`/`l3_baseline_drift`) and surfaced, not silently absorbed into the result.
106. The golden baseline is blessed right after a fresh install — explicitly via `dash l3-baseline capture --on <setup> …`, or auto-captured from the first flow's pre-state with a loud note that it assumes a fresh install. It stores only a state hash + the module/user summary — never secrets.
107. Restore is hybrid: service-enable changes are reverted via the `enable/disableService` mutation (which triggers the real rebuild), and any remaining `userdata.json` fields are file-restored; the restore reads the backend's `.onion` live.
108. What snapshot/restore cannot cover is listed on the run (`l3_state_uncovered`) and logged — service DB/app content, a created user's home dir + password hash until the next rebuild, backups already pushed to a provider, DNS records created at a provider, and a disabled service's torn-down unit when no Tor mutation channel was available. No silent caps.
109. Per-deploy local state (keepass secret DBs, L3 golden baselines) is git-ignored; the raw `userdata.json` is used only in-memory for restore and is never recorded or persisted.

## Integrity — hidden build inputs (Track 2, item 2, req 91)
110. The clean-state check also catches build-affecting files that `git status` cannot see: a catalog `guard_paths` file (e.g. users.nix, .env, .envrc, flake.local.nix, local.nix, secrets.nix) that is gitignored AND present in a repo under test is recorded (`guard_files`, with a content sha) and warned. A tracked guard file is already in the committed state, and an untracked one already trips the dirty gate — so only the gitignored+present (truly invisible) case is surfaced.
111. A self-contained nixosTest VM build must be hermetic: if the host nix `sandbox` is not `true`, or the build command passes `--impure`, the run is refused (recorded `hermetic`/`hermetic_problems`, with `nix_sandbox`/`impure`). Bypass only with `--allow-dirty` (then recorded as forced). This catches host state leaking into a build that is supposed to reflect only the committed code.

## Integrity — full build-closure identity (Track 2, item 3, req 92)
112. The recorded state identity is the exact build closure, not only the three input pins (api/nixpkgs/manager). A self-contained test (L1/L2) records its derivation path (`drv`/`closure`, via `nix eval <attr>.drvPath` — which captures every transitive input); a box run records the box's running-system store path (`closure`). A short hash of the closure is folded into `state_id` so two builds that differ only in a transitive input cannot collide on one identity.
113. The matrix surfaces the closure per run (in the run-history drawer) and flags closure divergence: more than one distinct build closure across runs on the same code-state key means a transitive input changed (same commits, different actual build). The sidebar grouping stays by repo-commits so a code state still shows all its tests together.

## Harness UX — fail fast before I/O
114. Args-only preconditions are rejected before any git fetch / lock read / SSH: a no-op entry (@todo/@manual) or an L3 run whose args can't work (no --on; a box setup missing --ip/--key/--token; chutney on a box; an android client not yet supported) fails in milliseconds, instead of paying per-entry base-building (notably `repos_behind_upstream(fetch=True)`'s `git fetch`). The authoritative box checks (reachability, live onion/domain, stamp identity) still run for valid args.
