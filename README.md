# Node Ninja Unreal Engine Skills

Use Node Ninja and Epic's Unreal Editor tools together from Claude Code. This fork extends Unreal Engine Skills with guidance for choosing capabilities, reading the installed Node Ninja guides, and combining native edits with analysis, patterns, and procedural workflows.

**Node Ninja includes RCTools.** Existing toolsets, classes, and resources retain their installed names until the planned rename. The skills discover those names rather than assuming `rct` identifiers have already become `nn` identifiers.

This repository contains instructions and a project-detection hook. It does not bundle the Unreal plugins or a static tool catalogue. It derives from Unreal Engine Skills for Claude Code by Thomas Mansencal / Epic Games; the upstream MIT notice is retained in [LICENSE](LICENSE).

## Contents

### Skills

- **`unreal-mcp`** (`skills/unreal-mcp`) - use Node Ninja and Epic tools through the routes exposed by the running editor; includes a focused [Node Ninja reference](skills/unreal-mcp/references/nodeninja.md).
- **`unreal-skill`** (`skills/unreal-skill`) - author project/plugin Agent Skills for workflow knowledge that discovery alone cannot supply.
- **`create-toolset`** (`skills/create-toolset`) - author tools while reusing existing capabilities across Epic and Node Ninja.

### Hooks

- **SessionStart** (`hooks/unreal-context.sh`) - injects a short note identifying the repo as an Unreal Engine project so Claude defaults to UE conventions (C++/`UObject`, Slate, UHT) and prefers the `unreal-mcp` skill. Runs on `startup`, `resume`, and `clear`.

## Prerequisites

1. **Unreal Editor** with **ModelContextProtocol** and **AllToolsets**, or the specific Epic toolset plugins needed for your work.
2. **Editor running to execute tools**, with the MCP server started. Run `ModelContextProtocol.StartServer`, or enable `bAutoStartServer` through `skills/unreal-mcp/references/setup.md`. The optional proxy can keep the client connection open while the editor is stopped.
3. **A bash shell on `PATH`** - required for the `SessionStart` hook (see **Platform Support**)
4. **Node Ninja / RCTools** enabled in the target Unreal project for its capabilities. Builds with the Epic integration register categories such as `RCTools.Blueprint` through Epic's ToolsetRegistry. The direct Node Ninja MCP connection supplies its native tools and guide resources. See [Node Ninja setup](skills/unreal-mcp/references/setup.md#5-node-ninja--rctools).

## Platform Support

Supported: **macOS**, **Linux**, and **Windows via Git Bash or WSL**.

The `SessionStart` hook is a bash script (`hooks/unreal-context.sh`) invoked by `hooks/hooks.json` as `bash "${CLAUDE_PLUGIN_ROOT}/hooks/unreal-context.sh"`. It requires a working `bash` on `PATH`:

- **macOS / Linux:** works out of the box.
- **Windows:** install **Git for Windows** (provides Git Bash) or run Claude Code under **WSL**. Native PowerShell without one of those does not have `bash` on `PATH`, and the hook will not run. The plugin's MCP tools still work; you just lose the short project-context note the hook injects at session start. There is no separate PowerShell companion script today.

## Installation

This fork has its own plugin identity, `nodeninja-unreal-engine-skills`. The upstream marketplace package does not install these Node Ninja extensions.

### Load this checkout

Start Claude Code from your Unreal project's directory, pointing at this checkout:

```powershell
claude --plugin-dir "C:\path\to\nodeninja-unreal-engine-skills"
```

On other machines, substitute the absolute path to this fork. `--plugin-dir` loads the plugin for that session. Use this fork in place of the upstream skill package for the project to avoid competing copies of the same skills. This does not install or configure either Unreal MCP server.

### Keep it loaded for one project

On Claude Code versions supporting skills-directory plugins, put this checkout at `<Project>/.claude/skills/nodeninja-unreal-engine-skills/` (including its `.claude-plugin/plugin.json`), or link that directory to an existing checkout. For example, from the project directory on Windows:

```powershell
New-Item -ItemType Directory -Force .claude/skills
New-Item -ItemType Junction -Path .claude/skills/nodeninja-unreal-engine-skills -Target 'C:\path\to\nodeninja-unreal-engine-skills'
claude plugin list
claude plugin details nodeninja-unreal-engine-skills@skills-dir
```

Use an unused link path; do not replace an existing package. Confirm the package is listed as a loaded project skills-directory plugin and its details include all three skills and the SessionStart hook. Start a fresh chat or reload plugins after registration or updates. Reading a `SKILL.md` file manually is not proof that the client loaded the package. This project-local registration does not affect other projects. See Claude Code's [plugin creation documentation](https://code.claude.com/docs/en/plugins/create).

## Verification

1. Launch Unreal Editor, then run `ModelContextProtocol.StartServer` in the console to start the MCP server.
2. Check the Output Log for MCP server startup messages.
3. In Claude Code, run `/plugin` and confirm `nodeninja-unreal-engine-skills` is loaded.
4. Run `/mcp`. Check for `unreal-mcp`, or `unreal-mcp-proxy` if you installed the proxy. A connected proxy does not prove that Unreal is reachable.
5. Try: "List all actors in the current level".
6. For Node Ninja, discover its categories in Epic's toolset listing or its direct connection, read a relevant guide if available, and make a read-only domain query. Neither a cached catalogue nor this skill package proves that the Unreal plugin is running.

## Configuration

Epic's default port is **8000** with URL path `/mcp`. If the port is in use, run `ModelContextProtocol.StartServer <port>` in the console with a different port number. A direct Node Ninja connection has its own configured endpoint; do not overwrite it with Epic's URL.

> **Note:** This plugin does not ship a static `.mcp.json` file. Run `ModelContextProtocol.GenerateClientConfig ClaudeCode` in the editor console to generate it from the current server port and URL; re-run after changing either.

### Optional proxy for editor recovery

If your engine build includes `Engine/Plugins/Experimental/ModelContextProtocol/Extras/Proxy`, you can use its `unreal_mcp_proxy` executable.

The proxy keeps the client-facing MCP session open while Unreal is unavailable and reconnects when Unreal starts again. It stores Unreal's native tool catalog as a temporary fallback. Cached tools do not prove that live calls can succeed. Without a cache, it exposes only `unreal_mcp_status` until the client fetches the recovered catalog.

See [proxy setup](skills/unreal-mcp/references/setup.md#4-optional-install-the-proxy) for platform binaries and installation commands. See [proxy recovery](skills/unreal-mcp/references/operations.md#proxy-recovery) for status, cached catalogs, and connection failures.

**Tool search** is enabled by default: the MCP server exposes three meta-tools (`list_toolsets`, `describe_toolset`, `call_tool`) instead of the full tool catalog, so Claude discovers toolsets on demand, the prompt cache stays warm, and discovered tools are callable on the same turn through `call_tool`. Toggle with the `bEnableToolSearch` setting in `[/Script/ModelContextProtocolEngine.ModelContextProtocolSettings]`; when disabled, every tool is registered upfront (path used by the hash-mapping commandlet). The model-facing usage contract lives in `skills/unreal-mcp/SKILL.md`.

## Security

Configured Unreal MCP connections give the assistant broad, live editor access. This instruction package does not install or enable those servers. Treat their execution access like running code from an assistant.

**Localhost is not a trust boundary.** The MCP server binds to `localhost:8000` with origin validation. Origin validation protects against a browser tab talking to the server, but any process running as the same user on the same machine can connect. Do not run the MCP server on shared or untrusted machines, and do not expose the port outside the loopback interface.

**`ProgrammaticToolset.execute_tool_script` executes arbitrary Python** inside the editor process. That script has full access to every toolset API, the project on disk, the asset database, and editor-privileged functions. Treat every invocation as a privileged operation that can mutate, move, or delete project content, and expect it to succeed without a second confirmation when approvals are disabled.

**`--dangerously-skip-permissions` removes the per-tool approval gate.** In that mode Claude can drive the editor and run Python through `execute_tool_script` with no further consent. A bad prompt in that mode can reach into the project and modify a large amount of state before you notice. Prefer not to run Claude Code with `--dangerously-skip-permissions` while this plugin is loaded; if you do, keep the blast radius small (narrow prompts, read-heavy tasks, a throwaway sandbox project).

**Source-control hygiene.** MCP tools edit live `UObject` state and can mutate, move, or delete VCS-tracked assets in a single call. Save and commit (or shelve) before any long MCP-driven session so the working copy is recoverable if Claude produces an unexpected result. Review the diff before submitting.

## What's Available

After the editor plugins and connections are configured, discover the tools supported by that installation. Epic's toolsets cover domains including:

- **Actors and Scene** - spawn, transform, inspect, and delete actors; manage components and outliner folders
- **Blueprints** - create, edit graphs, add nodes, connect pins, manage variables, compile
- **Assets and Content** - find, load, save, move, duplicate assets; edit Data Tables, Curve Tables, String Tables
- **Materials** - author material graphs, create and configure material instances
- **Meshes and Textures** - inspect/edit static and skeletal meshes, LODs, collisions, Nanite, sockets, bones
- **Animation** - build Control Rigs, inspect State Trees and Behavior Trees
- **Sequencer** - create and edit Level Sequences, keyframe animation, manage cameras, Control Rig integration, FBX import/export
- **VFX** - author Niagara systems and Dataflow graphs
- **UI** - build UMG widget blueprints, automate Slate UI interaction
- **Gameplay** - manage gameplay tags, inspect GAS state, create Game Feature Plugins, edit physics assets
- **Testing** - discover, run, and inspect C++ automation tests with detailed results
- **Editor** - screenshots, camera control, actor/asset selection, content browser, log inspection
- **Scripting** - batch multiple tool calls into a single Python script execution

Node Ninja adds focused semantic context, native snapshot patterns, domain authoring and analysis, PCG cook/probe workflows, and dungeon/room construction. Exact coverage depends on the installed plugin; the live schemas and guides are authoritative.
