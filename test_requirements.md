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
