# cc-kit

Personal Claude Code plugins — skills, commands, agents and hooks, published as a
plugin marketplace.

Marketplace name: `zwergpro-cc-kit` (the name is global on each user's machine, so
it carries the handle prefix; the repo itself stays short).

## Install

```
/plugin marketplace add Zwergpro/cc-kit
/plugin install <plugin>@zwergpro-cc-kit
```

To iterate locally without pushing, add the marketplace from a filesystem path:

```
/plugin marketplace add /abs/path/to/cc-kit
```

## Layout

```
cc-kit/
├── .claude-plugin/
│   └── marketplace.json        # marketplace manifest: name, owner, plugins[]
└── plugins/
    └── <plugin>/
        ├── .claude-plugin/
        │   └── plugin.json     # plugin manifest; only `name` is load-bearing
        ├── skills/
        │   └── <skill>/
        │       ├── SKILL.md    # required; dir name == frontmatter `name`
        │       ├── references/ # loaded on demand, not upfront
        │       └── scripts/
        ├── commands/           # optional: /slash commands, one .md each
        ├── agents/             # optional: subagent definitions
        ├── hooks/hooks.json    # optional
        └── .mcp.json           # optional: MCP servers the plugin ships
```

A plugin holds many skills — `plugin:skill` is how users address them. One plugin
per topic, not one per skill.

## Adding a plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json`:

   ```json
   {
     "name": "<name>",
     "description": "...",
     "version": "1.0.0",
     "author": { "name": "Sergey Nesterenko" },
     "homepage": "https://github.com/Zwergpro/cc-kit",
     "license": "MIT"
   }
   ```

2. Register it in `.claude-plugin/marketplace.json` under `plugins[]`, with a
   relative path source:

   ```json
   { "name": "<name>", "source": "./plugins/<name>", "description": "..." }
   ```

3. Add skills under `plugins/<name>/skills/<skill>/SKILL.md`. The frontmatter
   `description` is the only text loaded into context for trigger decisions — it
   has to say both what the skill does and when to fire it ("Use when ...").
   Keep SKILL.md short and push detail into `references/` files the body points at.

## License

MIT
