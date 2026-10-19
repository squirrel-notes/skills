# Squirrel Notes skills

Agent skills for [Squirrel Notes](https://squirrelnotes.app), the privacy-first, end-to-end encrypted notes app with long-term memory for AI agents.

## `squirrel-notes`

Teaches an agent to use the Squirrel Notes MCP server well:
- start each session with `get_context`;
- find memories reliably, with rephrasing and a near-miss retry;
- save memories that will be found again, with the right category, `keywords` and `validUntil`;
- correct them without losing history (supersede, invalidate, conflicts);
- choose between a memory and a note.

```
skills/squirrel-notes/
├── SKILL.md              the core workflow, loaded when the skill applies
└── references/
    ├── searching.md      how search ranks, reading results, filters
    ├── writing.md        when and how to save a memory, keywords, decisions
    ├── correcting.md     supersede, invalidate, conflicts, history
    ├── notes.md          working with notes
    └── setup.md          configuring the MCP server
```

## Requirements

The Squirrel Notes MCP server, with a Pro account's API key:
- `@squirrelnotes.app/mcp` for word search;
- `@squirrelnotes.app/mcp-embeddings` for word and meaning search (recommended).

See [`references/setup.md`](skills/squirrel-notes/references/setup.md).

## Install

**Claude Code.** Copy the skill folder into your skills directory:

```sh
git clone https://github.com/squirrel-notes/skills.git
mkdir -p ~/.claude/skills
cp -r skills/skills/squirrel-notes ~/.claude/skills/
```

Use `.claude/skills/` inside a project instead to scope it to that project.

To install a specific release rather than the latest, clone its tag: `git clone --branch vX.Y.Z https://github.com/squirrel-notes/skills.git`.

**Claude Desktop (and claude.ai).** Skills are uploaded as a ZIP with the skill folder at its root:

```sh
git clone https://github.com/squirrel-notes/skills.git
(cd skills/skills && zip -r ../../squirrel-notes.zip squirrel-notes)
```

Then, in Claude, open **Customize → Skills**, choose **+ → Upload a skill** and pick `squirrel-notes.zip`. Skills need code execution, which is under **Settings → Capabilities** (on Team and Enterprise, an owner enables it in the organisation settings). See [Use skills in Claude](https://support.claude.com/en/articles/12512180-use-skills-in-claude).

The skill only guides the agent: Claude Desktop also needs the Squirrel Notes MCP server in its configuration (**Settings → Developer → Edit Config**), as described in [`references/setup.md`](skills/squirrel-notes/references/setup.md) and the [MCP server's README](https://www.npmjs.com/package/@squirrelnotes.app/mcp-embeddings). Uploading a newer version replaces the old one.

**OpenClaw.** Install the skill folder with the OpenClaw CLI:

```sh
git clone https://github.com/squirrel-notes/skills.git
openclaw skills install ./skills/skills/squirrel-notes
```

Or copy `skills/squirrel-notes/` into `~/.openclaw/skills/` (every agent) or `<workspace>/skills/` (one workspace). New sessions pick it up. Add the MCP server under `mcp.servers` in OpenClaw's configuration (or with `openclaw mcp add`), and keep `SQUIRREL_API_KEY` and `SQUIRREL_PASSPHRASE` in OpenClaw's secret storage rather than in the config file. See OpenClaw's [skills](https://docs.openclaw.ai/tools/skills) and [MCP](https://docs.openclaw.ai/tools/mcp) docs.

**OpenCode and other agents that read `SKILL.md` skills.** Copy `skills/squirrel-notes/` into the agent's skills directory (for OpenCode, `.opencode/skills/`).

## Versions

This repository is published automatically when a Squirrel Notes release changes the skill: the commit `Release vX.Y.Z` and the tag `vX.Y.Z` hold the skill that matches that release's MCP server. A release that does not change the skill adds no commit or tag. Changes made here directly are replaced by the next publish.

## Privacy

The skill contains no data. Everything it describes happens through the local MCP server, which decrypts your notes and memories on your machine. The Squirrel Notes servers only ever store ciphertext.
