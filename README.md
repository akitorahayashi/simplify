# skills-plugin

A minimal template for one GitHub-installable Agent Skills plugin marketplace.
The repository root is the marketplace root for Claude Code and Codex, and the
`plugin/` directory is the plugin root for Claude Code, Antigravity CLI, and
Codex. One shared `plugin/skills/` directory supplies every client; each
manifest carries only client-specific identity.

## Structure

```text
skills-plugin/
├── .claude-plugin/
│   └── marketplace.json                # Claude Code marketplace manifest
├── .agents/
│   └── plugins/
│       └── marketplace.json            # Codex marketplace manifest
├── plugin/
│   ├── skills/
│   │   └── example-skill/
│   │       └── SKILL.md                  # one directory per skill; name comes from SKILL.md frontmatter
│   ├── .claude-plugin/
│   │   └── plugin.json                   # Claude Code manifest
│   ├── .codex-plugin/
│   │   └── plugin.json                   # Codex manifest
│   └── plugin.json                       # Antigravity CLI manifest
└── README.md
```

The marketplace manifests expose the plugin in this repository by pointing to
`./plugin`. They are installation indexes only; the plugin body is not duplicated
under a `plugins/` directory.

`plugin/skills/` is shared across all three clients. Each manifest carries only
that client's identity; the skill body is never duplicated. Component files
(`skills/`, and later `hooks/`, `agents/`, `commands/`, `.mcp.json`) live at the
plugin root. Only `plugin.json` belongs inside `.claude-plugin/` and
`.codex-plugin/`.

## What each manifest requires

- Claude Code — skills under `plugin/skills/` are auto-discovered, so
  `.claude-plugin/plugin.json` needs no `skills` field. Metadata like `author`,
  `homepage`, `repository`, `license`, and `keywords` is optional.
- Codex — `.codex-plugin/plugin.json` declares `"skills": "./skills/"`.
- Antigravity CLI — `plugin.json` is a closed schema: only `name` (required,
  `^[a-zA-Z0-9-_]+$`) and `description` are valid. Skills are discovered from
  `skills/`; do not add other fields.

## Customize

1. Rename the repository to your plugin name.
2. Rename `plugin/skills/example-skill/` to your skill's name and rewrite its
   `SKILL.md`. The `name` frontmatter sets the invocation name; the
   `description` frontmatter is the sentence the agent reads to decide when to
   use the skill, so make it a specific trigger.
3. Replace `example-plugin` with your plugin name (kebab-case) in the plugin
   manifests and marketplace manifests. Replace `skills-plugin` with your
   repository or marketplace name, and replace `your-name` in the Claude Code
   and Codex manifests.
4. Add more skills as sibling directories under `plugin/skills/`. Group related skills
   in one plugin rather than splitting one plugin per skill.
5. Validate before distributing:

   ```bash
   claude plugin validate .
   claude plugin validate ./plugin
   agy plugin validate ./plugin
   ```

## Install

The repository root is the marketplace root for GitHub distribution. Replace
`owner/skills-plugin` with the published repository.

### Claude Code

Claude Code installs the plugin from this repository's marketplace:

```bash
claude plugin marketplace add owner/skills-plugin
claude plugin install example-plugin@skills-plugin
```

For local development, Claude Code can load the plugin root for the current
session with `claude --plugin-dir ./plugin`.

### Codex

Codex installs the plugin from this repository's marketplace:

```bash
codex plugin marketplace add owner/skills-plugin
codex plugin add example-plugin@skills-plugin
```

### Antigravity CLI

Antigravity stages the plugin from the path that it receives. The `plugin/`
directory holds both `plugin.json` and `skills/`.

```bash
agy plugin install ./plugin
```
