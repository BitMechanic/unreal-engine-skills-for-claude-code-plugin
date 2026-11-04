# Operations: Console Commands, Settings, Recovery

Read this when something is misbehaving with the MCP server (tools missing, port collision, stale tool registry), or when you need a non-default configuration.

## Console commands

Run these from the Unreal Editor console (`~`).

| Command                                              | Use it for                                                                                               |
|------------------------------------------------------|----------------------------------------------------------------------------------------------------------|
| `ModelContextProtocol.StartServer [port]`            | Start the MCP server. Pass a port to override the default (e.g. when 8000 is in use).                    |
| `ModelContextProtocol.StopServer`                    | Stop the server. Useful if the registry is in a bad state and you want a clean restart.                  |
| `ModelContextProtocol.RefreshTools`                  | Re-register every toolset. Run this after the user enables a new toolset plugin.                         |
| `ModelContextProtocol.GenerateClientConfig <client>` | Regenerate the per-client config file. Args: `ClaudeCode`, `Cursor`, `VSCode`, `Gemini`, `Codex`, `All`. |

## Tool-search mode

Epic's `tools/list` has two modes, controlled by the `bEnableToolSearch` UPROPERTY on `UModelContextProtocolSettings` (default `true`). This setting does not control the separate direct Node Ninja MCP service:

- **`True`** (default): `tools/list` returns `list_toolsets`, `describe_toolset`, and `call_tool`. Schemas are read on demand through `describe_toolset`; those members execute server-side through `call_tool` rather than being registered as new native tools in the client.
- **`False`**: every toolset tool is exposed as a native MCP tool. The client may still defer loading schemas through its own tool search. Used by the hash-mapping commandlet.

Override in the same `.ini` used for `bAutoStartServer`:

```ini
[/Script/ModelContextProtocolEngine.ModelContextProtocolSettings]
bEnableToolSearch=False
```

## Node Ninja discovery and mixed connections

Integrated Node Ninja builds register categories with Epic, currently as `RCTools.<Category>`. These are discovered through Epic's toolset listing; the direct Node Ninja connection also exposes native tools and resource guides. Use exact names from the installed build throughout the naming transition.

- **Node Ninja category missing in Epic:** check the plugin is enabled and has initialized in the target editor, inspect its startup log, and verify whether this build includes the integration. Use an already available direct Node Ninja operation when appropriate. Do not treat a missing adapter category as proof that the direct service is down.
- **Guide resources missing in Epic:** inspect the direct Node Ninja connection's resource list. Adapter tool registration does not move its resources into Epic's AgentSkillToolset.
- **Epic operation sent to Node Ninja fails:** direct Node Ninja MCP does not proxy Epic tools; use the operation's discovered Epic route.
- **References work on one connection only:** confirm both endpoints target the same editor/project and check the expected argument representation. Reacquire references after reconstruction instead of retrying the mutation on another connection.

Discovery failures and permission refusals are different. Do not use an alternate route to bypass a refused permission.

## Proxy recovery

With the optional proxy, the client connects to `unreal-mcp-proxy` instead of directly to `unreal-mcp`. The proxy keeps that client session open while it reconnects to Unreal. Do not replace it with a second direct connection during normal recovery.

- `unreal_mcp_status` reports whether the proxy holds an initialized upstream session. It does not test current reachability.
- Visible tools can come from a stored catalog. Verify live access with a read-only Unreal tool call.
- With no stored catalog and no upstream session, the proxy exposes only `unreal_mcp_status`. After recovery, it sends `notifications/tools/list_changed`.
- Clients must fetch `tools/list` again after that notification. A client that ignores it can keep the status-only catalog, or an older cached catalog. Once Unreal is reachable, use the client's reconnect or configuration-reload action if available. Otherwise, restart the client to obtain the catalog. This is a client refresh limitation, not the normal proxy recovery workflow.
- Killing the proxy closes the client's STDIO connection. `Transport closed` then needs client reconnection; starting an unrelated proxy process cannot repair that connection. Ask before stopping active proxies.

If calls still fail, check the editor's MCP startup log, the configured endpoint, and proxy stderr. With multiple editors, identify which process owns the configured port. A running editor alone does not prove that the proxy targets its MCP server. Follow the recorded `.mcp.json` entry and the engine's `Extras/Proxy/README.md` for your build's transport limits.

## Troubleshooting matrix

| Symptom                                               | What to do                                                                                                                                                                                                                                                                                                          |
|-------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Unreal tools are missing, or `list_toolsets` errors | Check for both `unreal-mcp` and `unreal-mcp-proxy` in the client configuration. Check the editor's MCP startup log and configured endpoint. For a proxy connection, follow **Proxy recovery** above before concluding that the editor is stopped. |
| Editor logs "Failed to listen on port"                | Another process holds the default port. Change `ServerPortNumber` in the per-user `EditorPerProjectUserSettings.ini` (see `setup.md`), or pass `-ModelContextProtocolPort=<port>` on the next launch; restart the editor, and re-run `ModelContextProtocol.GenerateClientConfig ClaudeCode` to refresh `.mcp.json`. |
| A toolset you expect (e.g. `NiagaraTools`) is missing | Run `ModelContextProtocol.RefreshTools`. If still missing, the toolset's plugin may not be enabled in the `.uproject`. Check there.                                                                                                                                                                                 |
| Tool calls hang or return errors                      | Editor may be busy compiling, loading a level, or in PIE. Wait and retry. For long compiles, prefer `LiveCodingToolset.CompileLiveCoding`. It returns when the compile actually finishes.                                                                                                                           |
| `AIAssistantToolset.GetDockedContext` returns empty   | The Claude Code tab must be docked inside an asset editor (Blueprint, Material, etc.) to provide docked context. If undocked, that tool has nothing to report.                                                                                                                                                      |
| Concurrent tool calls produce conflicting results     | MCP accepts concurrent requests and handles dispatch synchronization. Serialize dependent calls and mutations that affect the same state. Use parallel calls only for independent work when the tools support safe overlap.                                                                                         |
