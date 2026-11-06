# merge-queue-pilot-mok-3569

A synthetic, throwaway repository that tests whether GitHub's native merge queue fits one gate design. It
holds no product code, no secrets and no credentials, and it is archived when the pilot ends. Tracking
issue: MOK-3569 (private tracker).

## What it tests

A ruleset on `main` requires a merge queue and one commit status, `pilot/gate`, from GitHub Actions
(integration 15368). The gate that posts it always runs the workflow on `main`, never pull request code:

| Workflow | Trigger | Permissions | Job |
|---|---|---|---|
| `merge-group-relay.yml` | `merge_group: checks_requested` | none | runs `true` |
| `gate.yml` | `pull_request_target` (opened, synchronize, reopened, auto_merge_enabled) and `workflow_run` of the relay | `contents: read`, `statuses: write` | reads the commit's tree through the REST API as data and posts `pilot/gate` on that commit |
| `on-push.yml` | `push` to `main` | none | records the pushed SHA and who pushed it |
| `bot-pr.yml` | `workflow_dispatch` | `contents: write`, `pull-requests: write` | opens one pull request as `github-actions[bot]` |

For a pull request head the gate posts on `pull_request.head.sha`. For a merge group the relay's run
completes, `workflow_run` starts the gate from `main`, and the gate posts on `workflow_run.head_sha`, the
group commit.

## The gate's rules

The gate never checks out, builds or runs the commit it judges. It lists the tree and reads files through
the API, and fails when:

- a file named `FAIL` sits at the tree root;
- the `check` spec names a `require <path>` the tree lacks, or holds a line the gate does not read;
- a pull request head changes anything under `.github/` against `main` (read from `compare/main...<sha>`, so
  the verdict depends only on the commit it is posted on; 300 or more changed files fail closed). No queue
  entry may change a workflow, because a merge group runs the group commit's own workflow files;
- a merge group's branch is not `gh-readonly-queue/main/pr-<N>-<sha>`.

`lab/README.md` describes the one timing directive scenarios use.

## Safety

- Only branches in this repository are judged; the gate skips pull requests from forks.
- On a pull request head that already carries a verdict, a re-run does not post `pending` over it; it posts
  only its new verdict.
- Nothing from an event payload is interpolated into a shell script; the gate reads it with `jq`.
- Every third-party action is pinned to a commit.
- Every job declares its own least permissions; the workflow default is none.
