# RULES-OF / worktree-breathing

Rules governing how the aurora@aurora machine reaches every repo, branch, and
worktree as if they were lungs — always available, always synchronised.

**Epoch note**: First written 2026-08-31. DarienSirius + ottopoet-thesean
enrollment is pending; rules are written proactively.

---

## The fundamental rule

Every GitHub account that can push code on behalf of this machine must have:
1. A dedicated SSH keypair enrolled on that account
2. A `gh auth` session for that account's PAT
3. A named SSH host alias in `~/.ssh/config`

If any of these three is missing, the account is not breathing.

---

## Accounts (aurora-thesean swarm)

| Account | Type | Status | SSH host alias | Keyring slot |
|---------|------|--------|----------------|--------------|
| aurora-thesean | ghUSER | ACTIVE — breathing | `aurora.wordgarden.dev` (default) | aurora-agent-* PAT in `gh auth` |
| DarienSirius | ghUSER | PENDING — not yet enrolled | `github-darien` (target) | keychain slot TBD |
| ottopoet-thesean | ghUSER | PENDING — not yet enrolled | `github-ottopoet` (target) | keychain slot TBD |

---

## SSH config pattern

Each non-default account gets a named Host alias that overrides the identity file:

```sshconfig
# ~/.ssh/config

Host github.com
    IdentityFile ~/.ssh/id_ed25519_git
    User git

# Per-account aliases for multi-account pushes
Host github-darien
    HostName github.com
    IdentityFile ~/.ssh/id_ed25519_darien
    User git

Host github-ottopoet
    HostName github.com
    IdentityFile ~/.ssh/id_ed25519_ottopoet
    User git
```

Usage: `git remote set-url origin git@github-darien:DarienSirius/repo.git`

---

## gh auth pattern

Each account needs its own PAT stored in `gh auth`:

```bash
# Enroll a new account without disturbing active session:
gh auth login --hostname github.com --with-token <<< "$THE_PAT"

# Switch active account:
gh auth switch --user DarienSirius

# Verify:
gh api /user --jq .login
```

PAT scopes required: `repo`, `gist`, `read:org`, `workflow`

PAT lifetime: rolling — set expiry, rotate before expiry, ack rotation warning.

---

## git worktree pattern

Branches are checked out as worktrees, never via `git checkout` in the live tree.

```bash
# For any branch work in ~/_ (home repo):
git worktree add /tmp/home-branch-NAME tasks/branch-NAME

# For repo work in the canonical lattice path:
git worktree add ~/_/AS/{org}/_/AS/{repo}/_/AS/{branch}/repo {branch}
```

**The critical prohibition**: never run `git checkout <branch>` inside `~/_ `.
The home repo is a deep tree. A checkout wipes the live working tree and breaks
any agent whose CWD is below the changed paths.

---

## Worktree lifecycle

```
Create → work → commit → push → PR → merge → prune
                                              ↓
                               git worktree remove <path>
                               git branch -d <branch>
```

Worktrees in `/tmp/` are ephemeral (survive reboot only by convention).
Worktrees in the canonical lattice path (`~/_/AS/.../`) are durable.

---

## Account enrollment checklist

Before enrolling an account (requires the human — Niobe — to provide PAT):

- [ ] Generate SSH keypair: `ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_{account} -C "{account}@aurora"`
- [ ] Add public key to GitHub account (Settings → SSH keys)
- [ ] Add SSH host alias to `~/.ssh/config`
- [ ] Enroll PAT: `gh auth login --hostname github.com --with-token`
- [ ] Verify: `gh auth switch --user {account} && gh api /user`
- [ ] Note PAT expiry date — schedule rotation reminder

---

## What "breathing" means

A machine that breathes can:
- Clone any repo from any enrolled account without prompting
- Push to any enrolled account's repos with the right host alias
- Switch between accounts in the same shell session via `gh auth switch`
- Operate multiple worktrees simultaneously without conflict

A machine that is not breathing must ask the human for credentials every time.
The goal of this quest is to eliminate that dependency for the three core accounts.

---

## Known gaps (as of 2026-08-31)

- DarienSirius SSH key: not yet generated or enrolled
- ottopoet-thesean SSH key: not yet generated or enrolled
- `~/.ssh/config` aliases: only default aurora-thesean entry exists
- `gh auth` sessions: only aurora-thesean active
