---
name: unreal-skill
description: "Create, edit, or review an Unreal Engine Agent Skill exposed by the editor, including C++ or Python skill classes and Blueprint skill assets. Use for project/plugin workflow instructions, descriptions, and registration. Covers TheConstructor and Epic workflows. For implementing tools use create-toolset; for running existing editor workflows use unreal-mcp. Excludes client-side Claude Code skill files."
---

# Unreal Skill

You are authoring an Unreal Engine Agent Skill: a named, reusable bundle of instructions that packages essential knowledge the agent either doesn't know or needs to pay close attention to. A skill can provide general best practices in a domain, guidance on a specific workflow, or a mix of both.

The skill returns guidance to its caller. It does not create an agent, dispatch a task or evaluate a result; those are optional responsibilities of the surrounding client. Keep the instructions usable by a single agent as well as a delegated one.

## Principles

A good skill is:

**Novel**: The content should be things the agent doesn't already know or can't learn by using tools. If the agent could figure it out by calling a tool, don't put it in a skill.

**Collegial**: Write like you're briefing a knowledgeable colleague, not authoring documentation. Assume the reader understands Unreal. Give them what they need to act, not a full explanation of how everything works.

**Flexible**: Skills can include conceptual explanations, step-by-step instructions, or a mix. Use whichever form makes the guidance clearest for the task.

**Durable**: Don't embed property names, tool names, or other details that change over time. Skills that reference specific API names break silently when those names change.

**Agnostic**: Don't reference orchestration systems, role names, model names, or anything about how the agent is wired up. Skills should work regardless of the surrounding infrastructure.

**Parsimonious**: Every token costs context. Put them where they matter most and cut anything that isn't essential or evergreen. If a sentence wouldn't be missed, remove it.

## Before You Write

Work through these questions before writing anything.

**1. Does the skill already exist?** Call `ListSkills` via MCP to see all registered skills in the project, then `GetSkills` on anything relevant. If the capability is already covered, point the user to the existing skill rather than creating a new one.

For TheConstructor, also check its installed MCP guide resources. Those are maintained with the plugin and are not automatically Agent Skills. Reuse existing guidance; add an Agent Skill only for workflow knowledge that is missing. Discover identifiers at runtime so instructions remain useful across the planned TheConstructor rename.

**2. Choose the implementation path.** Reuse the owner's existing language and maintained instructions:

- **C++ class**: Use the existing integration module when the owner already supplies a C++ guide. TheConstructor's Animation, StateTree and Dungeon skills use this path without a second guide body or registry.
- **Python class**: Use the plugin's existing Python package when its instructions and toolsets live there. It is version-controlled with the plugin and registers when imported.
- **Blueprint skill asset**: Use for project-specific guidance that does not belong in a code plugin. `CreateSkill` creates a Blueprint derived from `UAgentSkill` and returns its generated-class path.

## Structure

Every Unreal skill has two fields.

**Description**: Loaded at discovery time, before the skill's full instructions are read. One or two sentences that clearly describe what the skill covers and when it applies.

**Instructions**: The skill's payload. The actual guidance the agent follows when the skill is active. Apply the principles above: focus on what tools can't teach and cut anything not essential.

The agent discovers tools at runtime through Epic's meta-tools (`list_toolsets`, `describe_toolset`, `call_tool`) or the client's native tool discovery, depending on the connection. Write instructions that assume the agent will find the tools it needs rather than naming a fixed toolset list.

## C++ Skills

Derive a reflected `UCLASS` from `UAgentSkill` using the installed `ToolsetRegistry/AgentSkill.h` contract. Set a non-empty `Description` in the constructor. Reuse the owner's guide from `FString GeneratePrompt_Implementation(const FString& InitialInstructions) const` rather than copying instructions into the class.

Keep the class in the existing plugin module and use its established ToolsetRegistry dependency. Epic's derived-class discovery finds it after the module loads, subject to the configured skill filters; no manual tool registration is needed. Keep Epic and the engine unchanged.

`GetDetails` passes the skill's stored instructions to `GeneratePrompt`; `InitialInstructions` is not the user's current request. Retrieval should return guidance, without searching or importing a pattern library, compiling, saving, or mutating assets. Explicit tool calls perform those actions. `ListSkills` may load skill classes, so do not promise globally load-free discovery.

Build through the project's normal process, then verify the exact path returned by `ListSkills` with `GetSkills`. Reuse existing discovery, guide-equality and retrieval-purity evidence when the implementation is unchanged.

## Python Skills

A Python skill is a `UAgentSkill` subclass defined in a Python file inside a plugin. Use this path for the plugin's existing Python skill implementation.

### Authoring

Decorate the class with `@agent_skill` and inherit from `unreal.AgentSkill`. The two fields from the Structure section map to class attributes:

- The class docstring becomes the `Description`. Keep it to one or two sentences.
- `instructions` is a class attribute string containing the guidance loaded when the skill activates.

```python
import unreal

from toolset_registry.agent_skill import agent_skill

_INSTRUCTIONS = (
    'Do X before Y, because Z.\n'
    'Always verify the result after performing the operation.\n'
)

@agent_skill
class MySkill(unreal.AgentSkill):
    """Provides guidance on doing X in Unreal Engine.
    Apply this skill whenever the user wants to accomplish X."""

    instructions = _INSTRUCTIONS
```

### Registration

Skills register themselves on import. Unless directed otherwise, place skill files in a `skills/` subfolder within the plugin's Python package, import each one explicitly in that subfolder's `__init__.py`, and ensure `init_unreal.py` imports the `skills` package.

### Reloading

After editing a Python skill, reload the plugin's package before verifying. The editor won't pick up changes otherwise. Enable Remote Execution in **Edit → Project Settings → Plugins → Python → Enable Remote Execution**, then run:

```bash
python Engine/Plugins/Experimental/ToolsetRegistry/Content/Python/toolset_registry/tests/reload_remote.py your_plugin
```

## UAsset Skills

A skill asset is a Blueprint derived from `UAgentSkill`. The discovery tools return its generated-class path and read its default object's guidance. Use this path for project-specific instructions that do not belong in a code plugin.

### Authoring

Use `AgentSkillToolset` via MCP. Start by calling `ListSkills` to see what exists, then either create or update:

**Creating**: Call `CreateSkill` with:
- `FolderPath`: Content Browser folder, e.g. `/Game/Skills/`
- `AssetName`: PascalCase name, e.g. `MyWorkflowSkill`
- `Description`: one or two sentences (see Structure above)
- `Details`: a `FAgentSkillDetails` with `Instructions`

**Updating**: Call `UpdateSkill` with:
- `SkillPath`: full path to the skill, e.g. `/Game/Skills/MyWorkflowSkill.MyWorkflowSkill_C`
- `Description`: revised description
- `Details`: revised `FAgentSkillDetails`

## Reviewing Your Work

Before handing off, verify the skill looks right by calling `GetSkills` on its path. Then read the description and instructions together as the agent will see them:

- **Description**: Does it clearly say when this skill applies? Would an agent reading only the description know whether to activate it for a given task?
- **Instructions**: Do they teach something the agent couldn't learn from the tools? Are they brief enough to be worth the context cost?
