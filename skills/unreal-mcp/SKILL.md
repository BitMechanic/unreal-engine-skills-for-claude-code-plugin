---
name: unreal-mcp
description: "Inspect, edit, and run Unreal Editor workflows using Node Ninja (including RCTools) together with Epic's MCP tools. Use for live assets, graphs, animation, PCG, dungeon rooms, patterns, and editor operations. Also applies to Unreal requests identified by .uproject, Content Browser, or asset names such as BP_, ABP_, and PCG_. Skip pure conceptual or documentation questions."
---

# Node Ninja and Unreal MCP

Use the connected Unreal Editor tools to inspect and change the user's project. **Node Ninja includes RCTools.** Existing installations can expose RCTools names, `rct` prefixes, and `rctools://` resources. Use the identifiers actually discovered; do not invent `nn` replacements ahead of the rename.

Discover capabilities from the current editor rather than memorizing a catalogue. For Node Ninja work, read [Using Node Ninja with Epic](references/nodeninja.md), then load only the relevant guides served by the installed plugin.

Epic's connection commonly appears as `unreal-mcp` or `unreal-mcp-proxy`; the direct Node Ninja connection may still have an RCTools name. Project-specific server names are valid. Confirm the intended editor/project before combining operations across connections. See [setup](references/setup.md) for configuration.

## Discover the tool and its execution route

Epic's default search mode advertises `list_toolsets`, `describe_toolset`, and `call_tool`. When the Node Ninja integration is present, its categories also appear there, currently as `RCTools.<Category>`. Those described members execute through `call_tool`; describing them does not register new native MCP tools in the client.

When you start work:

1. Use `list_toolsets`, then `describe_toolset` for the relevant categories. Skip the listing if their exact names are already known. Inspect the required action's schema before calling.
2. Through Epic's dispatcher, pass the discovered `toolset_name`, short `tool_name`, and matching `arguments` to `call_tool`. Top-level dispatch without `toolset_name` is only for directly registered server tools, never `call_tool` itself.
3. If tools are exposed directly instead (Epic search mode disabled, or direct Node Ninja MCP), use the client's tool discovery and call the native tool with its schema. Direct Node Ninja MCP does not forward calls to Epic tools. Do not change server mode merely to follow this skill.

The same Node Ninja operation may be available through both routes. Choose one for each operation; do not repeat a mutation through the other route because its response shape differs.

Missing tools do not prove that the editor is stopped. Check the configured `unreal-mcp` or `unreal-mcp-proxy` connection and follow `references/operations.md`. A proxy can stay connected while Unreal is unavailable, and a client can keep a stale catalog after recovery. `unreal_mcp_status` reports the proxy's session state, not current reachability. Verify recovery with a read-only Unreal tool call. For first-time configuration, follow `references/setup.md`.

## Safety rules

Follow these rules to protect live editor state.

- **Preserve recoverability.** Use an appropriate save, duplicate, or existing recovery point before bulk changes. Save the intended assets after validation. MCP edits are not always undoable, especially across compilation boundaries; a sequence of tool calls is not automatically one transaction.
- **Wait for compilation.** Follow the project's build convention. Where Live Coding is appropriate and available, discover its compile tool and wait for the result before dependent calls.
- **Control concurrent calls.** MCP accepts concurrent requests and handles dispatch synchronization. Asynchronous tools can overlap, and request order does not guarantee execution or completion order. Serialize dependent calls and mutations that can affect the same asset or editor state. Use parallel calls only for independent work when the tools support safe overlap. Multiple agents can use one editor, but their conflicting changes can cause adverse effects.
- **Read the result's contract.** Epic and Node Ninja return different shapes; not every successful tool has a success flag, and a void return is not proof of the requested behavior. Stop dependent operations on errors, poll pending jobs using their advertised status operation, and verify the resulting native state.
- **Mind PIE.** Editor-only tools (asset creation in particular) behave differently while Play-in-Editor is active. If a result looks wrong, check whether PIE is running and stop it if so.

## Project skills

A project or plugin can register **Agent Skills** for naming conventions, setup, or workflows. Discover `AgentSkillToolset` if available; its skill listing is separate from the toolset listing. Node Ninja's existing MCP guide resources are another source of workflow knowledge, not automatically registered Agent Skills.

Check for them the same way you discover tools, and do it whenever you start unfamiliar work in a project rather than just once:

1. If `AgentSkillToolset` is available, inspect its schema and use `ListSkills` through the discovered route. Load the relevant returned skill paths with `GetSkills`.
2. If the skill is absent, filtered out, or unavailable through this connection, discover Node Ninja's resource descriptions and read the relevant returned guide URI as described in [the Node Ninja reference](references/nodeninja.md). A native skill that already returned that guide does not need a second resource read.
3. If neither surface is available, use discovered tool schemas and applicable project instructions. Report a missing capability only when it blocks the task; do not change filters or server mode to bypass a refusal.

Apply relevant project conventions within the user's requested scope. Authoring these skills is covered by the `unreal-skill` companion below.

## Reference files

- `references/nodeninja.md`: combining Node Ninja and Epic tools, finding installed guides, and handling live assets, patterns, and batched operations.
- `references/setup.md`: first-time MCP server setup, client configuration, and optional proxy installation.
- `references/operations.md`: console commands, settings, proxy recovery, and troubleshooting for missing tools or failed calls.

## Companion skills

- **`create-toolset`**: use when authoring a new toolset or adding tools to an existing one. Covers design principles, C++ and Python conventions, registration, error handling, and testing.
- **`unreal-skill`**: use when creating, updating, or reviewing an Agent Skill. Covers what makes a good skill and how to structure one.
