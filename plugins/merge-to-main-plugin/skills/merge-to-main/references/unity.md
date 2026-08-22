# Unity details for merge-to-main

Read this only when Phase 1 detected `ProjectSettings/ProjectVersion.txt`. It backs the Unity rows in `SKILL.md` Phase 3 (verify) and Phase 5 (monitor).

## Resolve the Editor binary

Read the exact version from `ProjectVersion.txt`'s `m_EditorVersion:` line — never guess, never use "latest installed". A mismatched Editor version can silently reimport assets differently or refuse to open the project.

- **macOS:** `/Applications/Unity/Hub/Editor/<ver>/Unity.app/Contents/MacOS/Unity`
- **Windows:** `C:\Program Files\Unity\Hub\Editor\<ver>\Editor\Unity.exe`
- **WSL2:** the Editor is a **Windows** process — it cannot run inside the WSL2 Linux environment. Invoke the Windows binary from WSL2 and convert the project path with `wslpath -w`:
  ```
  "/mnt/c/Program Files/Unity/Hub/Editor/<ver>/Editor/Unity.exe" -projectPath "$(wslpath -w .)" ...
  ```
  If the repo lives on the WSL ext4 filesystem (not `/mnt/c/...`), Unity can only reach it through `\\wsl$\...` at heavy I/O cost. In that case prefer the CI fallback below over a slow local run.

## EditMode test invocation

This one command is both the compile gate and the test gate — Unity refuses to run tests if compilation fails first.

```
<Unity> -batchmode -nographics \
  -projectPath <project> \
  -runTests -testPlatform EditMode \
  -testResults Logs/editmode-results.xml \
  -logFile Logs/unity-editmode.log
```

- **Do not add `-quit`** alongside `-runTests` — it truncates the run before results are written.
- Exit codes: `0` = all tests passed, `2` = one or more tests failed, `3` = run itself failed (e.g. compile error).
- On failure, parse `Logs/editmode-results.xml` for which test(s) failed. On a compile failure (exit `3`), grep `Logs/unity-editmode.log` for `error CS`.

## Editor lock

Only one Unity process may hold a project's `Library/` folder. If the developer already has the Editor open, batchmode fails immediately with "Multiple Unity instances cannot open the same project." Treat this as a **skipped** check, not a failed one — report it as skipped in the Phase 4 summary and fall back to CI as the gate.

## Cold import cost

Never run the gate against a fresh clone or a `git worktree` — an empty `Library/` forces Unity to fully reimport every asset, which can take many minutes even on a small project. Always run in the existing warm working checkout. This is exactly why Phase 1's `git switch -c <branch> origin/main` (same working directory, not a new worktree) is the right branching move for Unity repos.

## Asset integrity checks (tier 0)

Run these over the change set from `git diff --name-only origin/main...HEAD`. All are plain `git`/shell and return in well under a second — run them before lint.

**1. Every added asset has its `.meta`, and no `.meta` is orphaned:**
```bash
git diff --name-only --diff-filter=A origin/main...HEAD -- 'Assets/*' | while read -r f; do
  case "$f" in *.meta) continue ;; esac
  [ -f "$f.meta" ] || echo "MISSING META: $f"
done
```
A missing `.meta` breaks the asset's GUID on the next machine that pulls it — prefabs and scene references silently detach.

**2. No unresolved merge conflict markers in Unity YAML:**
```bash
git diff --name-only origin/main...HEAD -- '*.unity' '*.prefab' '*.asset' | \
  xargs -I{} grep -l '^<<<<<<<' {} 2>/dev/null
```
A conflict marker left in scene/prefab YAML corrupts the asset — Unity may still "open" it while silently dropping the conflicted section.

**3. No generated/build directories in the diff:**
```bash
git diff --name-only origin/main...HEAD | \
  grep -E '^(Library|Temp|Obj|Logs|Build|UserSettings)/|\.(csproj|sln)$'
```
These are machine-generated per environment; committing them causes spurious diffs and can overwrite a teammate's local state on pull.

**4. No large binary added outside Git LFS:**
```bash
git diff --diff-filter=A --name-only origin/main...HEAD | while read -r f; do
  size=$(git cat-file -s "$(git rev-parse HEAD:"$f")" 2>/dev/null || echo 0)
  [ "$size" -gt 10485760 ] && ! git check-attr filter "$f" | grep -q lfs && echo "LARGE, NOT LFS: $f ($size bytes)"
done
```
Art/audio/video committed raw instead of through LFS bloats the repo permanently — history size is not fixable by a later `.gitattributes` fix alone.

## `.gitattributes` expectations

A healthy Unity repo's `.gitattributes` should have:
- LFS rules for binary asset types the project actually uses, e.g. `*.png filter=lfs diff=lfs merge=lfs -text`, and similarly for `.psd`, `.fbx`, `.wav`, `.mp4`, `.exr`, etc.
- `*.unity text eol=lf`, `*.prefab text eol=lf`, `*.asset text eol=lf` — prevents whole-file spurious diffs from line-ending drift across macOS/Windows/WSL2.
- `merge=unityyamlmerge` on the same YAML types if the project has [UnityYAMLMerge](https://docs.unity3d.com/Manual/SmartMerge.html) configured — enables semantic merges of scenes/prefabs instead of raw text merges, which otherwise corrupt them on any real conflict.

If any of this is missing on a Unity repo, flag it in the Phase 4 summary as a one-time setup gap — not a per-merge blocker, but worth calling out once.

## CI fallback

When the local Editor run is skipped (WSL2 without a viable Windows Editor path, project lock held, no matching Editor version installed):
- State in the Phase 4 summary, plainly: "Local EditMode run skipped (<reason>) — CI is the compile/test gate for this merge."
- In Phase 5, watch the CI run (GameCI or Unity Build Automation) the same way as any other auto-deploy: background `gh run watch` or polling, confirm it's green before considering the merge verified.
- Never present a skipped run as a pass in the confirmation gate — this is the one thing that would make Phase 4's summary misleading.
