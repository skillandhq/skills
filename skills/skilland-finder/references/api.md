# Skilland API and MCP tools

Base address: `https://skilland.app/api/v1`. Every request is a plain GET without a key or
cookies; answers are JSON with an `api_version` field. A full description is at
`https://skilland.app/api/v1/openapi.json`. The MCP server at `https://skilland.app/mcp` has one
tool per request below and answers with the same JSON.

Requests are limited per IP address. When an answer is HTTP 429, wait about a minute before the
next request.

## Search: `GET /skills`

| Parameter | Meaning |
|---|---|
| `q` | Words to search for, in any language |
| `lang` | `en` or `zh`: the language of titles and summaries |
| `agent` | Only skills this agent can install (an id from `/agents`) |
| `category` | A category slug, such as `office-productivity` |
| `limit` | 1 to 50 (default 20) |
| `cursor` | `next_cursor` from the previous page |

MCP: `search_skills` with `query`, `lang`, `agent`, `category`, `limit`, `cursor`.

Each item in `skills` has `slug`, `name`, `title`, `summary`, `text_lang`, `category`, `license`
(`spdx`, `tier`), `scan_status`, `has_scripts`, `hosted_zip`, `url` (its page) and `access`.

## One skill: `GET /skills/{slug}?lang=en|zh`

MCP: `get_skill` with `slug` and `lang`.

The fields this skill reads, inside `skill`:

| Field | What it holds |
|---|---|
| `texts.{lang}` | `title`, `summary`, `overview`, `examples` |
| `license.spdx`, `license.tier` | The license; `mirror` when Skilland hosts a zip, `link_only` when installs come from upstream |
| `capabilities` | What the automatic check found: a list of `{ id, names, descriptions, values, evidence }`; `names` and `descriptions` are by language; `values` are hosts, runtimes, variable names, files or tools; `evidence` gives file, line and a link. `[]`: instructions only. `null`: no check result yet |
| `review` | `kind` (`skilland` or `third_party`), `rules_version`, `checked_at` (ISO date), `rules_table_version`, `new_capabilities` (ids new in this version, or `null`) |
| `scan` | `status` (`passed`, `warning`, …) and counts of findings |
| `requirements` | `runtimes`, `network`, `secrets`, `allowed_tools` |
| `pending_update` | `true` when a newer upstream version still waits for review |
| `version.upstream` | `repo`, `commit`, `folder_url` |
| `version.content_hash` | The hash `skills-lock.json` records for this version |
| `install_source` | `archive` (`url`, `sha256`), `github` (`owner`, `repo`, `path`, `ref`, `skill`), `discovery` (`url`, `skill`) |
| `compatible_agents` | Agent ids the install steps support |
| `urls.pages.{lang}` | The skill's page |
| `access` | `free`, or `members` / `purchase` for premium skills (nothing to install without signing in) |

A taken-down skill answers HTTP 410 with `error.code` `gone`; an unknown one 404.

## Install steps: `GET /skills/{slug}/install`

| Parameter | Meaning |
|---|---|
| `agent` | An agent id, or `other` |
| `os` | `windows`, `macos` or `linux` |
| `scope` | `user` (every project) or `project` (the current folder) |
| `lang` | `en` or `zh`, for the install prompt |

MCP: `get_install_steps` with the same arguments plus `slug`.

The answer has `available`, `target_path` (where the folder goes, written for that system),
`options` (in the order the site offers them) and `archive` (`url`, `sha256`) when a zip is hosted.
Option types:

- `skills-cli`: `command`, an `npx skills add …` line for sh and PowerShell, and `requires`.
- `manual-copy`: `command`, a script that downloads, checks the SHA-256 and copies the folder;
  `from` is `archive` or `github`.
- `zip-upload`: `download_url` and `sha256`, for agents that take uploads.
- `install-prompt`: `prompt`, a message for agents that edit files themselves.

When `available` is false, `unavailable.reason` says why (`premium`, `no_install_method`,
`invalid_source`).

## Agents: `GET /agents`

MCP: `list_agents` with `lang`.

Each agent has `id`, `name`, `skills_cli_agent_id` (null when the skills CLI doesn't support it),
`install_methods`, `skill_directories.user` and `.project` (the first one is where skills go; `~/`
is the home folder, `%USERPROFILE%` on Windows), `notes` and `docs_url`.

## Plugin marketplace (Claude Code)

`https://skilland.app/marketplace.json` is a Claude Code plugin marketplace named `skilland`. It
answers 404 until it is switched on. Each entry in `plugins` installs one skill: `name` is the
skill's slug, `metadata.skilland.skill` its `name`, and `source` the zip with its `sha256` or the
upstream folder at a commit.
