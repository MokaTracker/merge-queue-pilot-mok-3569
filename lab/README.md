# lab/

Each scenario pull request adds its own file here, so stacked pull requests never touch the same path.

A line `group-delay-seconds: <n>` in a `lab/` file that a pull request adds or changes makes the gate wait
`n` seconds (at most 120) before it posts its verdict on that pull request's own merge group. The gate
reads it from the group's diff against its first parent, so the delay applies to that one entry only and
never to later groups. It is scenario timing, not a check: it keeps a group in flight long enough to stack
another group on top of it, to dequeue it, or to push to its pull request.
