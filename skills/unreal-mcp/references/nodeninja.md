# Using Node Ninja with Epic

Node Ninja includes the toolsets currently named RCTools. Tool, class, and resource identifiers will be rationalized before release. Discover the installed names; do not translate prefixes or assume aliases exist.

## Read the installed guidance

The direct Node Ninja MCP server supplies domain descriptions during initialization and workflow resources through MCP `resources/list` and `resources/read`. Discover those resources through the client's supported resource interface. Current URIs include `rctools://guides/` and `rctools://index/`; use the exact returned URI. Read the getting-started guide and only the domain guides needed for unfamiliar work. Read the patterns guide for capture, reuse, import, or programmatic batching.

These guides are maintained with the implementation. Keep API details there rather than copying a tool catalogue into this skill. If the connection or client does not expose resources, use its available schemas and relevant project instructions; do not invent a guide-reading tool or assume Epic's AgentSkillToolset contains these resources (the native Animation and StateTree skills below return their guide text, not MCP resources).

For gameplay Animation Blueprints, AnimGraphs, state machines, Chooser selection or Animation pattern reuse, discover Epic's native `ListSkills` and `GetSkills` operations. Select the installed Animation Authoring skill by its returned description and path, then read it before loading detailed schemas. It serves the maintained Animation guide; retrieval does not search or import the pattern library. If the skill is absent or filtered out, use the Animation guide/resource route above and the discovered tool schemas.

For StateTree assets (states, tasks and conditions, transitions, parameters and bindings), discover the same native operations and select the installed StateTree Authoring skill by its returned description and path, then read it before loading detailed schemas. It serves the maintained StateTree guide. Inspect results with Epic StateTreeTools; attaching a tree to a controller or running it is separate work. If the skill is absent or filtered out, use the StateTree guide/resource route above and the discovered tool schemas.

Use Epic `list_toolsets`, `describe_toolset` and `call_tool` with returned identifiers for the Animation and StateTree tools; ordinary K2 work remains with Epic BlueprintTools. Keep parameters and action lists in the served schemas and owner guide rather than duplicating them here.

## Choose the operation by capability

Epic and Node Ninja operate on native Unreal state. In installations with the integration enabled, Epic's toolset listing includes Node Ninja categories (currently `RCTools.<Category>`). The direct Node Ninja connection remains available for its native tools and resources. Confirm both connections target the same editor before sharing references between them.

| Task | Starting point |
|---|---|
| Ordinary Blueprint or PCG graph creation/editing | Epic's native owner toolset. Discover node types, pins, properties, and argument shapes before building. |
| Asset lookup, duplication, saving, or generic UObject properties | Discover the corresponding Epic asset/object operation; use Node Ninja's domain lookup where it provides needed context. |
| Semantic context, focused analysis, capture/import, PCG cook/probe jobs, or dungeon/room workflows | Discover the relevant Node Ninja category and read its installed guide. |
| Animation, Control Rig, Material, Niagara, StateTree, or another overlapping domain | Read the relevant guide and compare the available operations. Choose by the required behavior, not just the vendor or domain name. |

Some ordinary Node Ninja Blueprint/PCG authoring endpoints have been retired in favor of Epic's native tools. Their absence is not a reason to recreate them, infer a renamed alias, or fall back immediately to unrestricted Python. Discover the owning operation first. Keep managed dungeon room changes with the domain workflow that owns synchronization and generation.

## Combine tools around the same native asset

1. Resolve the intended asset or actor and inspect the relevant context. Carry exact returned references and paths into subsequent operations; labels and short names may be ambiguous.
2. Use the native owner for the edit, with Node Ninja analysis or domain operations where the task needs them. Describe tools on both sides before assuming their reference representations are interchangeable. Reacquire references after operations that reconstruct objects.
3. Verify native state after the edit. Compile Blueprints when required by the operation sequence; PCG has no Blueprint compile step. Cooking or evaluating a graph and saving its asset are separate actions.

For example, editing a Blueprint can combine Node Ninja's semantic context with Epic's native graph edits, followed by Blueprint compilation, focused readback, and saving. A PCG task can combine native graph edits with Node Ninja's cook/probe workflow when generated output needs verification. A simple inspection does not require this full sequence.

## Recognize dungeon content in a level

When Epic's actor search or selection reveals Node Ninja dungeon content, inspect its actual class and ownership before editing. Current examples include `ANNDungeonLayoutActor`, `ANNDungeonClusterVolumeActor`, dungeon-managed `ARCTRoomActor` instances and `ARCTRoomFeature` actors. A label or prefix alone is insufficient; account for Blueprint subclasses and discover installed names. Managed rooms expose their layout root, cluster and room identity through reflected properties. Features expose `OwningRoom`; follow actual ownership, since a selected feature is not itself a dungeon root.

Use the Dungeon toolset to describe the current layout and the relevant cluster/interior before choosing a mutation. Current entry points include `nn_dungeon_describe_layout` and `nn_dungeon_describe_cluster_volumes`; read their served schemas and preserve returned identities. Resolve a selected room or generated component back to its owner rather than treating it as the layout root. Layout data counts and completed PCG generation do not by themselves prove valid visible geometry or traversability.

Keep layout, cluster, room, doorway and managed generation changes in the Dungeon workflow. Use Epic's scene/object tools for ordinary level objects and the domain's supported presentation edits. Generic actor or generated-instance edits are not substitutes for updating authored dungeon state.

Room features are authored objects with explicit placement, not merely generated decoration. `nn_dungeon_get_cluster_room` can return the synchronized `room_ref` and inspect owned features; use that room reference for room operations, not the cluster host. `room_read_floor_ascii` returns the floor boundary, usable-floor mask and live feature occupancy, with exact feature references and floor-local poses in its legend. Use the existing floor-feature tools to preview/apply precise placement, then read back the room.

For objects on a feature, discover its tagged authored surfaces with `feature_list_surfaces`, then use `feature_read_surface_ascii` and the existing surface-prop tools. Carry the exact feature/surface identities and their local coordinate frame into placement. Positioned props are authored records with native mesh components, not necessarily independent gameplay actors. Read ASCII alongside structured geometry and diagnostics; surface occupancy does not test external blockers or establish placement safety. These room and surface workflows do not require a PCG graph edit.

Use Node Ninja PCG tools when the question concerns graphs or generated output. `pcg_list_level_components` supplies exact actor/component references and graph assignments; multiple PCG components can exist on one actor. Use focused graph context, live generated-instance reads and retained execution reports as needed. Instance indices and generated component references can change after regeneration; mesh observations do not establish logical room membership or collision validity.

Dungeon-managed room components permit observation but refuse external in-place cook and cleanup. Do not bypass that ownership through another execution route. If a fresh diagnostic capture is needed, follow the installed PCG guide: enable inspection on the exact idle component, let the Dungeon owner generate, read the retained result, then release that inspection interest. Merely discovering or inspecting the actor does not require generation. The current room guard is not proof that every dungeon actor family has the same guard.

## Reuse, capture, and import

For an independent copy within one project, start with native asset duplication. Use Node Ninja's pattern library when the task calls for capturing or importing a reusable snapshot. Search its metadata and prerequisites before importing; inspect the selected pattern rather than loading every candidate.

Keep live asset paths, saved pattern IDs, imported destination paths, and pending job IDs distinct. A pattern read does not inspect later edits to its source. Import success does not prove dependency reconnection, compilation, or behavior. Follow the owner guide for required plugins/types, compatible dependency bindings, import hints, and explicit refresh after edits. Do not infer automatic dependency copying or animation retargeting.

## Programmatic batching, when useful

Epic's programmatic executor can combine registered Epic and Node Ninja operations in a finite script when that reduces repeated calls or large intermediate results. Discover its execution-environment tool and read the current contract before writing the script. The installed patterns guide describes the Node Ninja result and error conventions.

Use separate tool calls for simple work. In a script, preserve the distinction between errors and pending jobs, return only the useful result, and stop dependent mutations on failure. There is no automatic batch rollback; inspect partial state before retrying. A tool deliberately excluded from the Epic adapter is not made available by batching, and another connection must not be used to circumvent a permission refusal.
