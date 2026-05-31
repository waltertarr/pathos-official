# Echoes of the Soul — GitHub & Local Setup

Concrete, copy-pasteable steps to fork Pathos, set up a buildable environment on this
machine, and run the day-to-day git workflow. Tailored to: Windows 11, PowerShell,
`gh` authenticated as **waltertarr**, .NET 9 installed.

> Nothing here has been executed yet — this is the runbook for when you approve moving past
> planning. Commands are written for **PowerShell**.

---

## Part A — Fork & branch the repo

The official README's workflow is: install Pathos → fork the repo → set up VS/VS Code → build.

```powershell
# 1. Fork callanh/pathos-official to your account AND clone it locally.
#    Cloned into a sibling of this planning folder.
cd C:\Users\walte\VsCodeApps
gh repo fork callanh/pathos-official --clone --remote

# This creates:
#   - GitHub repo:  waltertarr/pathos-official  (public fork)
#   - local clone:  C:\Users\walte\VsCodeApps\pathos-official
#   - remotes:      origin   -> waltertarr/pathos-official
#                   upstream -> callanh/pathos-official
```

```powershell
# 2. Create and switch to the feature branch.
cd C:\Users\walte\VsCodeApps\pathos-official
git switch -c feature/echoes-of-the-soul

# 3. Push the branch so it's publicly visible immediately.
git push -u origin feature/echoes-of-the-soul
```

That branch on the public fork **is** "the publicly available branch." Anyone can now see it at
`https://github.com/waltertarr/pathos-official/tree/feature/echoes-of-the-soul`.

> Decision (locked): keep the fork named `pathos-official` and do the work on the named
> branch — easiest path to pull upstream updates and (optionally) open a PR back later.

---

## Part B — Local build environment

Per the upstream README, building a variant requires the **installed Pathos desktop game**
plus a C# toolchain. .NET 9 is already present; the missing piece is Pathos itself.

### B1. Install Pathos desktop → `C:\Games\Pathos`

- Get "Pathos for Windows Desktop" (linked from the official site / repo README).
- Install to the default `C:\Games\Pathos`. The build tooling (`PathosMaker`) expects the
  installed game to load the generated `Pathos.Codex`.

Verify afterward:

```powershell
Test-Path C:\Games\Pathos        # expect: True
Get-ChildItem C:\Games\Pathos | Select-Object -First 10
```

### B2. Toolchain

- **Option 1 — VS Code** (lightweight): install the C# Dev Kit extension. Build with
  `Ctrl+Shift+B`, run with `F5` (per README).
- **Option 2 — Visual Studio 2026**: open the `.sln`, run the `PathosOfficial` project.

CLI build (sanity check the SDK can restore/build the solution):

```powershell
cd C:\Users\walte\VsCodeApps\pathos-official
dotnet --version                 # 9.0.306
dotnet restore
# Locate the solution, then build it:
Get-ChildItem -Filter *.sln
dotnet build .\<SolutionName>.sln -c Debug
```

> The exact solution/project names and whether `dotnet build` fully replaces `PathosMaker`
> are **unknowns until Phase 0**. Confirm the canonical build path (VS Code `Ctrl+Shift+B`
> vs `dotnet build`) during the audit and record it in `docs/ECHOES_REPO_AUDIT.md`.

### B3. First build BEFORE any changes

Build the untouched fork and launch it once. This proves the toolchain works and gives you a
known-good baseline to compare Echoes changes against. If this fails, fix the environment
before writing a single line of Echoes code.

---

## Part C — Day-to-day git workflow

```powershell
# Standard commit (small, scoped — follow the spec's 15-commit plan)
git add <paths>
git commit -m "feat(echoes): <what changed>"
git push                          # pushes to origin/feature/echoes-of-the-soul

# Keep the fork current with upstream (do periodically, not mid-feature)
git fetch upstream
git switch feature/echoes-of-the-soul
git merge upstream/main           # or: git rebase upstream/main
git push
```

Commit message convention (from the spec):

```
docs: ...                 design/audit/progress docs
feat(echoes): ...         new Echoes functionality
```

The spec's suggested 15-commit sequence is reproduced in
[ECHOES_EXECUTION_CHECKLIST.md](ECHOES_EXECUTION_CHECKLIST.md).

---

## Part D — Where the planning docs go

These planning files currently live in `C:\Users\walte\VsCodeApps\pathos-branch` (outside the
repo). When the fork is cloned:

- Copy **PATHOS_ECHOES_BRANCH_SPEC.md** → `pathos-official/docs/ECHOES_OF_THE_SOUL.md`
  (the spec's recommended top-level design doc). Commit it first (commit 1).
- Keep this `ECHOES_PLAN.md` / setup / checklist set either in `docs/` too, or leave them in
  the planning folder as your private working notes. Recommendation: put the spec + a trimmed
  progress doc *in the repo*; keep verbose planning notes out of the public history if you prefer.

---

## Part E — Licensing & attribution (do not skip)

Pathos is **non-commercial freeware**. As a variant author you must:

1. **Keep every upstream license/notice file intact** (do not delete or edit `LICENSE`,
   copyright headers, attribution).
2. **Never monetise** the variant in any form.
3. **Make the variant clearly a derivative** — `docs/ECHOES_OF_THE_SOUL.md` should credit
   upstream Pathos and link `callanh/pathos-official`.
4. Add a short `NOTICE`-style note (or a section in the README/docs) stating this is an
   unofficial non-commercial variant built on official Pathos content.

> Confirm the exact license file name/type during Phase 0 and record it in the audit doc.

---

## Part F — Sharing it

Because the fork is public, sharing = sending the branch URL:

```
https://github.com/waltertarr/pathos-official/tree/feature/echoes-of-the-soul
```

Optional later steps:
- Open a **draft PR** from your branch to `callanh/pathos-official` to surface the work
  (only if you intend to propose it upstream — not required to "make it public").
- Pin the branch / add a README badge / write a short announcement once the MVP is playable
  (the spec's "Shareable Branch Goal" defines what "playable enough to share" means).
