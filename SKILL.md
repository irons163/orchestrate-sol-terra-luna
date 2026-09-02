---
name: orchestrate-sol-terra-luna
description: "Keep GPT-5.6 Sol in the main thread as the lead orchestrator, delegate difficult but clearly bounded work to GPT-5.6 Terra Max, and assign clear, repeatable work to GPT-5.6 Luna Max. Use when the user requests Sol orchestration, Terra Max or Luna Max subagents, tiered multi-agent coding, parallel code review, module analysis, independent feature implementation, testing, debugging, or result integration. Check model and Max reasoning availability before starting. If any required capability is unavailable, stop all work and explain how to enable it; proceed only after availability is confirmed."
---

# Sol–Terra–Luna Orchestration

Keep Sol in the main thread to understand the objective, break down tasks, make architectural decisions, verify completion, and integrate results. Delegate difficult but clearly bounded work to Terra Max, and clear, repeatable, easily verifiable work to Luna Max.

## 1. Check capabilities first (the only permitted preliminary work)

Before interpreting, breaking down, or executing the user's substantive task, check the capabilities actually exposed by the current environment. Do not rely on model memory or assumptions:

1. When observable, confirm that the main thread uses `gpt-5.6-sol`. If it is known to use another model, apply the complete hard stop and ask the user to switch. If the main thread's model cannot be observed, do not block solely for that reason.
2. Confirm that subagents can explicitly select `gpt-5.6-terra` and `gpt-5.6-luna`.
3. Confirm that both models can explicitly select `max` reasoning effort.
4. If the tool declares support but an actual launch is rejected because of model or reasoning incompatibility, treat that capability as unavailable.
5. Proceed to Section 2 only after confirming the availability of Sol in the main thread (when observable), `Terra Max`, and `Luna Max`.

Do not use the App's Max visibility setting in the model picker as a capability check. That setting only controls whether Max appears in the composer; `model_reasoning_effort = "max"` in a custom agent file directly specifies the spawned session's reasoning effort. Use the subagent tool declarations and actual launch results as the criteria.

### Any required capability is unavailable: complete hard stop

If Sol in the main thread, `Terra Max`, `Luna Max`, or any required model/reasoning combination is unavailable, immediately stop the entire workflow, not just delegation. Apart from checking capabilities and explaining how to enable them, do not run tools or advance the substantive task. This includes:

- Do not read or inventory the project, code, documents, or external data.
- Do not perform architecture design, compliance boundary analysis, requirements breakdown, risk analysis, or test planning.
- Do not create substitute subagents or let Sol proceed alone with main-thread work that could otherwise be done safely.
- Do not downgrade to `high`, `xhigh`, or another model/reasoning configuration on your own.

Do not introduce exceptions such as "This does not prevent Sol from first working on..." While this skill is active, a missing required capability is a complete stopping condition. Use a different workflow only if the user explicitly cancels this skill or explicitly changes the required model combination.

The only permitted setup exception: if the user explicitly asks the AI to help enable the required capabilities, the AI may guide them through the App settings. If desktop control is available, it may also open the Codex App settings and select the required options directly. The AI may also create, inspect, and validate Luna Max or Terra Max custom agent configurations in `~/.codex/agents/` or `.codex/agents/`. This exception does not permit reading the original project, analyzing the original task, or advancing any main-thread work.

### Required response when Luna Max is unavailable

State precisely whether `gpt-5.6-luna`, Luna's `max` reasoning, or both are missing. Then provide the following steps and end the response:

1. In the Codex desktop App, locate the model/reasoning controls below the input box.
2. Open **Advanced** and check whether `gpt-5.6-luna` can be selected. If Max is visible, it can be checked manually here. If the model picker hides Max, do not conclude that Luna Max is unavailable for that reason alone; continue checking the custom agent configuration, subagent tool declarations, and actual launch results.
3. If the main thread was switched to Luna for this check, switch back to `gpt-5.6-sol` afterward. Luna must run as a subagent and must not replace Sol in the main thread.
4. Treat the App's Max visibility setting as optional. Only if the user wants to select Max manually in the main task's composer, open **Settings** (macOS: Cmd+,; Windows: Ctrl+,) and enable Max under **Configuration** or the model/reasoning settings. If the user explicitly requests AI assistance and desktop control is available, select the option for them; otherwise, ask them to do it manually. An unchecked Max option must not trigger a hard stop by itself and does not affect an explicit `model_reasoning_effort = "max"` setting in a custom agent file.
5. If Luna is absent from Advanced, check the account plan, the workspace administrator's model policies, and whether the current provider offers `gpt-5.6-luna`. Explain clearly that the skill itself cannot unlock a model that is not provided.
6. If Luna and Max are available to the account but no dedicated subagent has been defined, instruct the user to create `~/.codex/agents/luna-max.toml` (personal) or `.codex/agents/luna-max.toml` (project) with the following content:

```toml
name = "luna_max"
description = "Dedicated Luna Max worker for clear, bounded, repeatable tasks."
model = "gpt-5.6-luna"
model_reasoning_effort = "max"
developer_instructions = """
Handle only clear, bounded, repeatable tasks. Stay within the assigned scope, verify the result, and return concise evidence to the Sol main thread.
"""
```

This configuration only specifies the subagent's model and reasoning effort. It does not grant model access that the account does not already have.

7. Return to this task and retry after setup. If the capability list has not refreshed, open a new task and invoke `$orchestrate-sol-terra-luna` again.

Choose the closing message based on the configuration state rather than always repeating the same line:

- **The AI has not yet created or validated the configuration**: use this final line:

  > When finished, reply: **Luna Max is enabled**. If you would like AI assistance with setup, reply: **Please help me configure Luna Max.**

- **The AI has created and validated `luna-max.toml`, but the current task has not reloaded the capabilities**: do not ask the user to enable Max in the App first or repeat the setup request. Use this final line instead:

  > The Luna Max configuration file is ready. If the capability list has not refreshed, reply: **Please duplicate the current task and test Luna Max.**

After successfully creating and validating the configuration, do not ask the user again to reply "Please help me configure Luna Max."

Maintain the hard stop until both `gpt-5.6-luna` and `max` are confirmed available in the current environment.

### Terra Max is unavailable

Apply the same hard-stop principle, adapting the Luna steps above for `gpt-5.6-terra`. Do not interpret "supports Max" as "necessarily supports Terra or Luna," and do not substitute models unless the user cancels this skill.

## 2. Map the work

Sol first defines the overall objective, constraints, acceptance criteria, and dependencies, then routes tasks according to the nature of the work:

| Role | Assigned work |
| --- | --- |
| Sol main thread | Ambiguous requirements, architectural and cross-module decisions, task breakdown, conflict resolution, final acceptance, and integrated output |
| Terra Max | Difficult but self-contained module analysis, nontrivial independent implementation, in-depth code review, security or concurrency reasoning, and complex root-cause investigation |
| Luna Max | Code search and fact gathering, test execution and bug reproduction, log classification, mechanical edits, small features with precise specifications, and structured summaries |

Do not delegate trivial tasks merely to use subagents. If core requirements remain unclear, Sol should resolve them first instead of relying solely on increased subagent reasoning effort.

## 3. Delegate with an explicit contract

Every subagent prompt must include:

- **Objective**: describe one independently achievable outcome.
- **Context and inputs**: provide the necessary files, symbols, errors, or specifications.
- **Scope**: list the modules or files the agent may read and write.
- **Restrictions**: identify interfaces, behaviors, and adjacent work that must not be changed.
- **Completion criteria**: define verifiable acceptance criteria.
- **Validation**: specify tests, type checks, lint checks, reproduction steps, or evidence.
- **Report format**: request a summary, changed files, validation results, risks, and unresolved questions.

Use a hub-and-spoke structure by default: Sol delegates directly to Terra and Luna, and both report directly to Sol. Do not let Terra manage Luna unless Terra has been authorized to take full ownership of an independent subsystem.

## 4. Control concurrent writes

Parallelize only independent work. When multiple agents share a working directory, assign exclusive file or module ownership to each writing task. If scopes overlap, run the tasks sequentially or use isolated worktrees.

Suitable cross-checking patterns:

- Terra implements a complex feature, Luna creates or runs focused tests, and Sol performs the final integration.
- Luna completes a clearly specified change, Terra performs an in-depth review, and Sol decides whether to accept it.
- Luna gathers reproduction and log evidence, Terra analyzes the root cause, and Sol decides on the cross-module fix.

## 5. Verify and integrate

After all required results are available, Sol:

1. Checks whether subagents stayed within scope and met their completion criteria.
2. Directly inspects important diffs, file references, tests, and reproduction evidence rather than accepting summaries alone.
3. Resolves conflicting conclusions using code and validation results.
4. Performs integration validation appropriate to the level of risk.
5. Reports the completed work, validation results, remaining risks, and the actual division of work among Sol, Terra Max, and Luna Max in the final response.

Do not treat an agent's own claim of completion as final approval. Final responsibility remains with Sol in the main thread.
