# Sol–Terra–Luna Orchestrator

A Codex skill that keeps GPT-5.6 Sol in the main thread as the lead orchestrator, while delegating independent subtasks to Terra Max and Luna Max based on the nature of the work.

## Role assignments

| Role | Responsibilities |
| --- | --- |
| GPT-5.6 Sol | Understand the objective, break down tasks, make architectural decisions, review results, and integrate the final output |
| GPT-5.6 Terra Max | Handle difficult but clearly bounded analysis and implementation, in-depth code review, and complex debugging |
| GPT-5.6 Luna Max | Perform clear, repeatable, and easily verifiable searches, tests, reproductions, mechanical edits, and summarization |

## Core behavior

- Verify that Sol, Terra Max, and Luna Max are available before starting substantive work.
- If any required model or Max reasoning capability is unavailable, stop completely instead of substituting another model or using a lower reasoning level.
- Once all capabilities are available, delegate subtasks with explicit objectives, scope, completion criteria, and validation methods.
- Sol reviews important diffs, tests, and evidence, and is responsible for the final integration.

## Requirements

- Codex supports skills, custom agents, and subagents.
- The main thread can use `gpt-5.6-sol`.
- Subagents can use `gpt-5.6-terra` and `gpt-5.6-luna`, and both support `max` reasoning effort.

Whether Max is selected in the Codex App model picker only affects whether the option is displayed in the composer. It does not affect the explicit `model_reasoning_effort = "max"` setting in custom agent files. The actual subagent startup result is the criterion for determining whether the capability is available.

## Installation

Run the following on macOS or Linux:

```bash
git clone https://github.com/irons163/orchestrate-sol-terra-luna.git "${CODEX_HOME:-$HOME/.codex}/skills/orchestrate-sol-terra-luna"
```

If the current task does not reload the skill, create a new task. If it still does not appear, restart the Codex App.

## Usage

Explicitly enable the skill in a Codex prompt:

```text
$orchestrate-sol-terra-luna
```

You can also describe the work directly, for example:

```text
Use the Sol–Terra–Luna Orchestrator to review this project: Sol handles integration, Terra Max performs the architecture and security review, and Luna Max runs tests and organizes the errors.
```

If Luna Max has not been configured, you can ask:

```text
Please help me configure Luna Max.
```

After configuration, if the current task has not reloaded the agent capabilities yet, you can ask:

```text
Please duplicate the current task and test Luna Max.
```

## Repository structure

```text
.
├── README.md
├── SKILL.md
└── agents/
    └── openai.yaml
```

Personal `luna-max.toml` and `terra-max.toml` files are not included in the repository. The skill provides instructions for creating and validating them based on the user's environment.
