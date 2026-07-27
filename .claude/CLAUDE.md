# Personal Harness Rules

## Git

- Never commit or push changes to a repository automatically. Committing and
  pushing are always the user's call — only do either when the user explicitly
  asks in the current conversation, whether via a normal prompt or by invoking
  a skill whose purpose includes committing/pushing. This applies to any
  subagent or workflow you spawn on the user's behalf too — they must not
  commit or push without the same explicit ask.

## Load on this machine

- Ask before running anything that will noticeably load the machine, and wait
  for an answer. Say what it will do, roughly how heavy it is and for how long,
  then let the user decide. This machine often has several projects running at
  once — builds, emulators, dev servers, other agents — and none of that is
  visible from inside a single session. Things that need asking: deliberate CPU
  or memory saturation (stress tests, `stress-ng`, busy loops), high job counts
  (`-j$(nproc)`, large `--concurrency`), full test suites run in a loop,
  emulators or simulators, and anything expected to run for many minutes.
- Prefer the lighter way of getting the same answer. A slow machine can usually
  be simulated with a timeout, a fake clock, an injected delay, or a couple of
  niced workers — reach for saturating every core last, not first.
- Anything spawned must be cleaned up, and the cleanup verified rather than
  assumed. Background jobs outlive the shell that started them, so a trailing
  `kill` in the same command is not enough on its own. Bound the work up front
  (`timeout`, a self-limiting loop), then confirm afterwards with `pgrep`/`ps`
  that nothing is left running, and say what was checked. Never report a
  cleanup as done on the strength of a command having printed something.
- This applies to any subagent or workflow spawned on the user's behalf.
