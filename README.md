# testresults — SelfPrivacy-over-alternative-nets test result store

Separate from the code repos on purpose: results are keyed by the **code repo's commit SHA**
(`repo` + `commit` in each record), so recording a result never dirties or moves the code
commit — you can run many tests back-to-back on the same state.

- `results.jsonl` — one JSON record per run (appended). Read by `dev-dashboard/matrix.html`.
- `logs/<run_id>.log` — full output per run.
- `media/<run_id>.shots/` — step screenshots (committed).
- **videos are NOT committed** (`.gitignore`) — they stay local artifacts under the runner's
  `dev-dashboard/media/`, viewable only on the machine that recorded them.

Written/read by `dev-dashboard/dash` (which clones this repo into `dev-dashboard/testresults/`).
