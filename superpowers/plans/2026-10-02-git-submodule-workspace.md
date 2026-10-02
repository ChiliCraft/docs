# ChiliCraft Nested Repositories Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use `executing-plans` to implement this plan task-by-task with verification checkpoints.

**Goal:** Nest all eight ChiliCraft plugin repositories and the documentation repository in the existing `ChiliCraft/chilicraft` parent repository, with repeatable initialization and an intact aggregate build.

**Architecture:** Keep `chilicraft` as the Gradle aggregator and replace each existing `cc-*` source directory with a Git submodule at the same path. Rename `chilicraft-meta` to `docs`, flatten its current `docs/` contents into its repository root, then mount it at the parent’s existing `docs/` path. Add shell and PowerShell wrappers around Git’s recursive submodule initialization.

**Tech Stack:** Git submodules; Bash; Windows PowerShell; Gradle Kotlin DSL; GitHub Actions; JDK 21.

## Global Constraints

- Register exactly these nine child repositories: `cc-core`, `cc-survival`, `cc-demon`, `cc-soul`, `cc-martial`, `cc-adventure`, `cc-season`, `cc-quest`, and `docs`; do not nest `chilicraft` inside itself or invent a `cc-street` repository.
- Preserve all current parent files and all contents unique to the split repositories. The eight module source trees were verified against GitHub `main`; overlapping source-file blobs match, with each `build.gradle.kts` the sole differing overlapping blob and nine extra files in each split repository.
- The parent remains uncommitted and unpushed. The user authorized renaming, committing, and pushing only the documentation repository as part of preparing `ChiliCraft/docs`.
- Do not modify Java sources, add dependencies, start a Minecraft server, or alter unrelated user files.
- Keep standalone module builds intact. The parent build must compile the checked-out `:cc-core` project instead of requiring GitHub Packages.
- Use JDK 21 and the project’s existing Gradle wrapper; successful `build` is the acceptance criterion.

---

### Task 1: Prepare and publish the documentation submodule

**Files:**
- Remote rename: `ChiliCraft/chilicraft-meta` → `ChiliCraft/docs`
- Modify in the documentation repository: `README.md`, `AGENTS.md`, `.github/workflows/docs.yml`, `architecture.md`, `events-protocol.md`, `performance.md`
- Move in the documentation repository: `docs/architecture.md`, `docs/copywriting-requests.md`, `docs/development-plan.md`, `docs/events-protocol.md`, `docs/module-dev-guide.md`, `docs/performance.md`, `docs/superpowers/`
- Preserve/add: `docs/superpowers/specs/2026-10-02-git-submodule-workspace-design.md`, `docs/superpowers/plans/2026-10-02-git-submodule-workspace.md`

**Interfaces:**
- Produces a public repository at `https://github.com/ChiliCraft/docs.git`, with its documentation files at repository root and the parent-compatible `docs/superpowers/` tree.
- The parent mounts this repository at `docs/`; therefore `architecture.md` and related links must continue to work from the parent layout.

- [ ] **Step 1: Check rename preconditions and authorized GitHub access**

Run:

```bash
gh auth status
gh repo view ChiliCraft/docs --json name
```

Expected: authenticated account has permission to rename `ChiliCraft/chilicraft-meta`; the second command reports that `ChiliCraft/docs` does not already exist. If the target exists or permission is missing, stop before changing the remote and request a resolution.

- [ ] **Step 2: Rename the documentation repository and clone its new URL**

Run:

```bash
gh api -X PATCH repos/ChiliCraft/chilicraft-meta -f name=docs
workdir="$(mktemp -d /private/var/folders/60/0btch0dj3111k_1grpv_9xmw0000gn/T/opencode/chilicraft-docs.XXXXXX)"
git clone https://github.com/ChiliCraft/docs.git "$workdir/docs"
```

Expected: the GitHub API response names the repository `docs`, and the clone checks out `main` without authentication errors.

- [ ] **Step 3: Flatten the existing documentation directory without dropping repository-only files**

In the clone, use `git mv` for the six listed Markdown files and `docs/superpowers` into the repository root. Keep `README.md`, `AGENTS.md`, `.gitignore`, `.github/`, the two root-level reference documents, and every other existing file. Copy the approved design and this implementation plan into the clone’s `docs/superpowers/specs/` and `docs/superpowers/plans/` before moving that tree.

Update repository references from `ChiliCraft/chilicraft-meta` to `ChiliCraft/docs` in `README.md` and `AGENTS.md`. Update repository-relative references in those files from `docs/<file>` to `<file>`, and change their inventory descriptions from a nested `docs/` directory to the documentation repository root. Update the project-structure section in `architecture.md` to show the parent `ChiliCraft/chilicraft` root, all eight same-path module submodules, and the `docs/` submodule. Update `.github/workflows/docs.yml` so its presence check tests `architecture.md`, `events-protocol.md`, `performance.md`, and `module-dev-guide.md` at repository root. In `events-protocol.md` and `performance.md`, change the two `../cc-*` source links to absolute links into `ChiliCraft/cc-core` and `ChiliCraft/cc-survival`, so the documents also resolve when viewed in the standalone docs repository.

Run:

```bash
git -C "$workdir/docs" status --short
git -C "$workdir/docs" diff --check
```

Expected: only the intended documentation moves and path updates appear; no files outside the documentation repository are changed.

- [ ] **Step 4: Verify documentation paths, then commit and push only the docs repository**

Run the relocated docs workflow’s shell checks locally:

```bash
test -f "$workdir/docs/README.md"
test -f "$workdir/docs/AGENTS.md"
test -f "$workdir/docs/architecture.md"
test -f "$workdir/docs/events-protocol.md"
test -f "$workdir/docs/performance.md"
test -f "$workdir/docs/module-dev-guide.md"
```

Set the clone-local commit email to `2822603942@qq.com` with `git -C "$workdir/docs" config user.email "2822603942@qq.com"`; use the already configured Git author name. Review `git -C "$workdir/docs" diff --check` and `git -C "$workdir/docs" status --short`, then run:

```bash
git -C "$workdir/docs" add README.md AGENTS.md .github/workflows/docs.yml .gitignore architecture.md copywriting-requests.md development-plan.md events-protocol.md module-dev-guide.md performance.md superpowers "ChiliCraft 开源插件缝合候选调研报告.md" "ChiliCraft·插件版技术文档（实现规格 v1.0） (1).md"
git -C "$workdir/docs" commit -m "chore: flatten documentation repository"
git -C "$workdir/docs" push origin main
```

Expected: only `ChiliCraft/docs` receives this documentation-layout commit; the parent repository is not committed or pushed.

### Task 2: Replace the eight existing module directories with submodules

**Files:**
- Create/update parent `.gitmodules`
- Replace parent paths: `cc-core/`, `cc-survival/`, `cc-demon/`, `cc-soul/`, `cc-martial/`, `cc-adventure/`, `cc-season/`, `cc-quest/`
- Replace parent path `docs/` with the published `ChiliCraft/docs` submodule

**Interfaces:**
- Each path is a Git submodule pinned by the parent gitlink to a commit on the corresponding repository’s `main` branch.
- The documentation submodule URL is `https://github.com/ChiliCraft/docs.git`.

- [ ] **Step 1: Confirm the parent is safe to migrate and create a narrow backup directory**

Run:

```bash
git status --short --untracked-files=all
git diff --check
backup="$(mktemp -d /private/var/folders/60/0btch0dj3111k_1grpv_9xmw0000gn/T/opencode/chilicraft-parent-backup.XXXXXX)"
```

Expected: no pre-existing user changes in the eight module directories. Keep the approved design and plan changes in `docs/`; preserve them in the docs repository prepared in Task 1. Keep `$backup` until all parent checks pass.

- [ ] **Step 2: Convert each module path while retaining its original directory outside the worktree**

For each name in this exact list — `cc-core cc-survival cc-demon cc-soul cc-martial cc-adventure cc-season cc-quest` — run these commands with that name substituted for `<repo>`:

```bash
git rm -r --cached -- <repo>
mv <repo> "$backup/<repo>-monorepo"
git submodule add "https://github.com/ChiliCraft/<repo>.git" <repo>
```

Expected: `git submodule add` checks out the split repository at the same path, `.gitmodules` records its URL, and the original parent directory remains available under `$backup` until final verification.

- [ ] **Step 3: Convert `docs/` to the prepared documentation submodule**

Run:

```bash
git rm -r --cached -- docs
mv docs "$backup/docs-monorepo"
git submodule add https://github.com/ChiliCraft/docs.git docs
```

Expected: `docs/architecture.md`, `docs/events-protocol.md`, `docs/performance.md`, the other current documentation files, and the approved design/plan files are present through the new submodule. No `docs/docs/` path is created.

### Task 3: Keep aggregate builds and initialize submodules uniformly

**Files:**
- Modify: `build.gradle.kts`
- Modify: `.github/workflows/build.yml`
- Modify: `README.md`
- Create: `scripts/init-repos.sh`
- Create: `scripts/init-repos.ps1`

**Interfaces:**
- Parent Gradle builds resolve `com.chilicraft:cc-core` to `project(":cc-core")`; independent child builds retain their current Maven resolution.
- Both scripts run `git submodule update --init --recursive` from the parent root and propagate Git’s exit status.

- [ ] **Step 1: Add parent-only dependency substitution**

Add this block to the root `build.gradle.kts` after the existing `subprojects` configuration:

```kotlin
allprojects {
    configurations.configureEach {
        resolutionStrategy.dependencySubstitution {
            substitute(module("com.chilicraft:cc-core"))
                .using(project(":cc-core"))
        }
    }
}
```

Expected: modules that declare `compileOnly("com.chilicraft:cc-core:1.0.0")` in their independent repositories resolve against the included local core when built from the parent.

- [ ] **Step 2: Make parent CI check out every pinned submodule**

Change the existing checkout step in `.github/workflows/build.yml` to:

```yaml
      - name: 检出仓库
        uses: actions/checkout@v4
        with:
          submodules: recursive
```

Expected: CI receives all nine gitlink targets before running the existing JDK 21 `build dist` steps.

- [ ] **Step 3: Add the Bash initialization entry point**

Create `scripts/init-repos.sh` with:

```bash
#!/usr/bin/env bash
set -euo pipefail

script_dir="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)"
repo_root="$(git -C "$script_dir" rev-parse --show-toplevel)"

git -C "$repo_root" submodule update --init --recursive
```

Set its executable bit. Expected: it works when called from any current directory inside or outside the checkout.

- [ ] **Step 4: Add the Windows PowerShell initialization entry point**

Create `scripts/init-repos.ps1` with:

```powershell
$ErrorActionPreference = "Stop"
$repoRoot = (Resolve-Path (Join-Path $PSScriptRoot "..")).Path

git -C $repoRoot submodule update --init --recursive
if ($LASTEXITCODE -ne 0) {
    exit $LASTEXITCODE
}
```

Expected: it initializes from the script’s parent directory and propagates a nonzero Git exit status.

- [ ] **Step 5: Document clone and initialization commands in the parent README**

Add a short “Clone and initialize repositories” section with these exact commands:

```bash
git clone --recurse-submodules https://github.com/ChiliCraft/chilicraft.git
```

For an existing clone, document:

```bash
bash scripts/init-repos.sh
```

and the Windows PowerShell equivalent:

```powershell
.\scripts\init-repos.ps1
```

Explain that `cc-*` are independent repositories pinned by the parent, and module changes must be committed in their child repository before the parent gitlink is updated.

### Task 4: Verify repository state, initialization, and build

**Files:** All parent changes above; no additional files.

- [ ] **Step 1: Check whitespace, submodule registration, and parent scope**

Run:

```bash
git diff --check
git submodule status --recursive
git status --short --untracked-files=all
```

Expected: `.gitmodules` lists exactly the eight module URLs and `docs`; every submodule status has a checked-out commit (no leading `-`); parent status contains only the intended `.gitmodules`/gitlinks, build, CI, README, and script changes.

- [ ] **Step 2: Exercise initialization twice and check Bash syntax**

Run:

```bash
bash -n scripts/init-repos.sh
if command -v pwsh >/dev/null 2>&1; then
  pwsh -NoProfile -Command '$tokens = $null; $errors = $null; [System.Management.Automation.Language.Parser]::ParseFile((Resolve-Path "scripts/init-repos.ps1"), [ref]$tokens, [ref]$errors) | Out-Null; if ($errors.Count -gt 0) { $errors | Format-List; exit 1 }'
fi
bash scripts/init-repos.sh
bash scripts/init-repos.sh
git submodule status --recursive
```

Expected: both invocations succeed; the second makes no repository-content changes; every submodule remains initialized at the parent-pinned commit.

- [ ] **Step 3: Build the parent aggregate without starting a server**

Run:

```bash
sh gradlew build
```

If and only if dependency metadata refresh or network/TLS access fails while dependencies are already cached, run:

```bash
sh gradlew build --offline
```

Expected: `BUILD SUCCESSFUL` for the parent and included modules. Do not run Minecraft.

- [ ] **Step 4: Preserve the requested uncommitted parent result**

Recheck `git status --short --untracked-files=all` and `git diff --check`. Do not commit or push the parent. Remove only the exact temporary backup directory created in Task 2 after confirming the parent build and all nine submodule checkouts succeed; if any validation fails, retain it for recovery.

Expected: final parent modifications remain available for the user to review, while the documentation repository alone has the authorized pushed commit.
