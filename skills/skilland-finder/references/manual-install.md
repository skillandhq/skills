# Installing a skill by hand

For method 3 in SKILL.md: no Node.js, or an agent the skills CLI doesn't support. From the install
steps you need `archive.url`, `archive.sha256`, `target_path` and the skill's `name`.

The install steps' `manual-copy` option already holds a script for the user's system that does
everything below (download, SHA-256 check, copy). Showing that script to the user and running it
after the user agrees is the shortest route. The commands here are for doing the same yourself.

A Skilland zip holds one folder named after the skill (`<name>/SKILL.md`), sometimes with
`LICENSE`, `NOTICE` and `SKILLAND-SOURCE.md` added to it. Install that folder's contents into
`target_path`, so that `<target_path>/SKILL.md` exists.

## Windows (PowerShell 5.1 or 7)

Write `%USERPROFILE%` in `target_path` as `$HOME`, PowerShell's name for the same folder. Run the
lines in order and stop at the first error.

```powershell
$url = '<archive.url>'
$sha = '<archive.sha256>'
$dest = Join-Path $HOME '.claude\skills\<name>'
$tmp = Join-Path ([IO.Path]::GetTempPath()) ([guid]::NewGuid().ToString())
New-Item -ItemType Directory -Path $tmp | Out-Null
Invoke-WebRequest -UseBasicParsing -Uri $url -OutFile (Join-Path $tmp 'skill.zip')
if ((Get-FileHash (Join-Path $tmp 'skill.zip') -Algorithm SHA256).Hash -ne $sha) { throw 'SHA-256 mismatch: nothing was installed' }
Expand-Archive -Path (Join-Path $tmp 'skill.zip') -DestinationPath (Join-Path $tmp 'x')
$src = Join-Path $tmp 'x\<name>'
if (-not (Test-Path (Join-Path $src 'SKILL.md'))) { $src = Join-Path $tmp 'x' }
New-Item -ItemType Directory -Force -Path $dest | Out-Null
Copy-Item -Path (Join-Path $src '*') -Destination $dest -Recurse -Force
Remove-Item -Recurse -Force $tmp
```

## macOS and Linux

Write `~` in `target_path` as `$HOME`. On Linux, use `sha256sum -c -` instead of
`shasum -a 256 -c -`.

```sh
(
  set -e
  url='<archive.url>'
  sha='<archive.sha256>'
  dest="$HOME/.claude/skills/<name>"
  tmp="$(mktemp -d)"
  curl -fsSL "$url" -o "$tmp/skill.zip"
  echo "$sha  $tmp/skill.zip" | shasum -a 256 -c -
  unzip -q "$tmp/skill.zip" -d "$tmp/x"
  src="$tmp/x/<name>"
  [ -f "$src/SKILL.md" ] || src="$tmp/x"
  mkdir -p "$dest"
  cp -R "$src/." "$dest/"
  rm -rf "$tmp"
)
```

## When the checksum doesn't match

Stop. Delete the temporary folder, install nothing and tell the user that the download didn't match
the SHA-256 Skilland published. Don't retry from another address.

## Skills without a hosted zip

When `archive` is null, Skilland only links to the skill. Use `install_source.github` (`owner`,
`repo`, `path`, `ref`): download `https://codeload.github.com/<owner>/<repo>/zip/<ref>`, unzip it
and copy `<the zip's top folder>/<path>` into `target_path`. There is no checksum to compare. When
`ref` is a 40-character commit it is the commit Skilland checked; otherwise tell the user that the
files may be newer than what Skilland checked, and prefer the skills CLI.

## Skill folders of common agents

The first folder for each scope, from Skilland's agent list. Prefer `target_path` from the install
steps, or `skill_directories` from `/agents`: they follow the current list.

| Agent | Every project (`user`) | This project (`project`) |
|---|---|---|
| Claude Code | `~/.claude/skills` | `.claude/skills` |
| OpenAI Codex | `~/.agents/skills` | `.agents/skills` |
| Cursor | `~/.cursor/skills` | `.agents/skills` |
| GitHub Copilot (VS Code) | `~/.copilot/skills` | `.agents/skills` |
| Gemini CLI | `~/.gemini/skills` | `.agents/skills` |
| Antigravity | `~/.gemini/config/skills` | `.agents/skills` |
| WorkBuddy | `~/.workbuddy/skills` | `.workbuddy/skills` |
| Any other agent | `~/.agents/skills` | `.agents/skills` |
