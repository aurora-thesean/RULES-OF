# RULES-OF / agent-entry-points / auto-loaded-context

Rules governing how AI agent harnesses discover and auto-load their entry-point
context files. These rules are ground truth — agents depend on them to know what
they will and will not automatically see when they wake.

**Epoch note**: First written 2026-08-17. Prior aurora@aurora epoch (Epoch 0, WOGAJI
period) predates this documentation. Isomorphic lattice reconstruction is ongoing.

---

## The fundamental rule

Context auto-loads from the directory where the session **starts**.
Not from directories the agent navigates to during the session.
The start directory is the dungeon entrance. Everything else is exploration.

---

## Per-harness rules

### Claude Code (aurora)

**Auto-loaded file**: `CLAUDE.md`
**Location**: the working directory when the session is invoked
**Scope**: start directory only — walking to a subdirectory does NOT load that
directory's CLAUDE.md into the current session's context

**Include mechanism**: `@filename` in CLAUDE.md causes Claude Code to auto-include
that file's content, enabling cross-harness context from a single entry point:
```
# In CLAUDE.md:
@AGENTS.md
@AGENTS.override.md
```

**BIAFRAL doctrine**: Each directory SHOULD have a CLAUDE.md as its warrant.
This does not mean it auto-loads — it means a Claude agent starting there will
find it. Directories without a warrant are incomplete.

---

### Codex

**Auto-loaded file**: `AGENTS.md`
**CLAUDE.md**: ignored
**Override mechanism**: If `AGENTS.override.md` is present, Codex reads it instead
of AGENTS.md. AGENTS.md is silenced. This allows environment-specific overrides
without modifying the shared AGENTS.md.

---

### GitHub Copilot agents

**Auto-loaded files**: BOTH `CLAUDE.md` AND `AGENTS.md`
**Priority**: implementation-defined; treat both as active

---

### Summary table

| File | Claude Code | Codex | Copilot |
|---|---|---|---|
| `CLAUDE.md` | ✅ auto-loaded | ❌ ignored | ✅ auto-loaded |
| `AGENTS.md` | ⚠️ BIAFRAL only | ✅ auto-loaded | ✅ auto-loaded |
| `AGENTS.override.md` | ❌ no effect | ✅ mutes AGENTS.md | unknown |
| `@filename` includes | ✅ works | ❌ no | ❌ no |

---

## The isomorphic lattice

The local filesystem structure at `~/RULES-OF/` mirrors this repository exactly.
When this repo is cloned to `~/RULES-OF/`, the local and remote structures are
identical (isomorphic). Agents writing to local paths should commit to the
matching remote path.

Violations (local-only or remote-only structure) are tech debt. Finding one
means writing the mirror side.

---

## What this means for directory design

A directory that multiple harnesses will start agents in should have:
```
./CLAUDE.md           — Claude Code entry point (include @AGENTS.md here)
./AGENTS.md           — Codex + Copilot entry point
./AGENTS.override.md  — (optional) Codex override for this environment
```

A directory that only Claude Code agents start in:
```
./CLAUDE.md           — sufficient
```

A directory in BIAFRAL territory (subdirs agents walk to, not start in):
```
./CLAUDE.md           — warrant only; won't auto-load but documents the space
./AGENTS.md           — same
```

---

## Known gaps (Epoch 0 reconstruction)

The aurora@aurora machine ran an undocumented epoch (WOGAJI period) before these
rules were written. Many directories have AGENTS.md or CLAUDE.md files from that
period whose accuracy against these rules is unverified. Reconstruction proceeds
by reading existing files, checking against these rules, and correcting.
