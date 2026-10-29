# pnpm repeat install leaves a missing `workspace:` link missing

Minimal reproduction for pnpm 12.16.1:
after a declared `workspace:` dependency's link is removed from `node_modules`,
a repeat `pnpm install` prints `Already up to date`, exits 0, and does not recreate the link.
`pnpm install --force` does.

The workspace has two projects.
`app` depends on `lib` through `workspace:*`.
No registry access is needed.

## Steps

```bash
pnpm install --offline
rm app/node_modules/lib
pnpm install --offline
ls app/node_modules/lib
pnpm install --offline --force
ls app/node_modules/lib
```

## Expected

The second `pnpm install --offline` recreates `app/node_modules/lib`.

## Actual

Recorded on 2026-10-29 with pnpm 12.16.1, default settings:

```text
step 1: install exit 0 | app/node_modules/lib: symlink -> ../../lib
        output: Scope: all 3 workspace projects / Already up to date / Done in 135ms using pnpm v12.16.1
step 2: after rm: absent
step 3: install exit 0 | link: absent
        output: Scope: all 3 workspace projects / Already up to date / Done in 20ms using pnpm v12.16.1
step 4: install --force exit 0 | link: symlink -> ../../lib
```

The first install, which does create the link, also ends with `Already up to date`,
so that message does not tell the two cases apart.

## Where it comes from

In pnpm commit `f7d7e6173ad68ee44918173bdded12602966cb80`,
the repeat-install fast path treats a link that exists but points nowhere as stale
and treats an absent entry as healthy:
[`modules_dirs.rs` lines 164 to 168](https://github.com/pnpm/pnpm/blob/f7d7e6173ad68ee44918173bdded12602966cb80/pnpm/crates/package-manager/src/optimistic_repeat_install/modules_dirs.rs#L164-L168)
and [lines 205 to 207](https://github.com/pnpm/pnpm/blob/f7d7e6173ad68ee44918173bdded12602966cb80/pnpm/crates/package-manager/src/optimistic_repeat_install/modules_dirs.rs#L205-L207).

---

Written by an agent (Claude Code, claude-sonnet-5-5).
The steps were run once by the agent as written above; no human has re-run them yet.
