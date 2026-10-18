# ROTOM prompt templates

This folder provides the finalized prompts for direct inference, the orchestrator, workers, and aggregator, including the textual game description given to every model. It covers the agents used in live development and final ROTOM evaluation. Earlier preliminary and exploratory offline prompts are outside this package.

| File | Contents |
| --- | --- |
| [prompt_templates.json](prompt_templates.json) | All four role templates, shared instructions, information layout, game/role checklists, dynamic construction rules, output formats, and nine complete observed GLEE task descriptions. |
| [example_prompts.json](example_prompts.json) | One fully constructed request for each role, including its input observation and response schema. |

## Models and inference settings

Final ROTOM routes Bargaining to Lite Direct, Negotiation to homogeneous three-worker orchestration, and Persuasion to Terra Direct. The live development comparison also evaluated heterogeneous orchestration. The exact model identifiers and recorded generation settings are:

| Model identifier | Use | Thinking or reasoning setting |
| --- | --- | --- |
| `gemini-3.5-flash-lite` | Lite Direct; all homogeneous orchestrator, worker, and aggregator calls; heterogeneous orchestrator, aggregator, and one worker | `thinking_level="medium"`; no numeric thinking budget supplied |
| `gpt-5.6-terra` | Terra Direct | `reasoning_effort="medium"` |
| `gpt-5.6-luna` | One heterogeneous worker | No reasoning parameter supplied; provider generation parameters were `{}` |
| `deepseek-v4-flash` | One heterogeneous worker | `{"thinking": {"type": "disabled"}}` |

Gemini uses the Google GenAI API. Terra, Luna, and DeepSeek use OpenAI-compatible chat-completions interfaces. Temperature, output-token caps, and top-p were not explicitly supplied. No fixed seed was supplied. Gemini automatic function calling was disabled.

Base per-call timeouts were 30 seconds with up to two transport retries for Gemini, Luna, and DeepSeek, and 60 seconds with up to one retry for Terra. Remaining decision time can shorten these budgets. The reasoning settings above were verified against recorded requests as well as the configured profiles.

## Prompt construction

Each request contains three parts, in order:

1. **Shared information:** game family, role, phase, round, opponent, textual game description, exposed configuration, visible state, public history, deterministic calculations, checklist, and legal actions.
2. **Shared decision instruction:** strategic guidance and the own-payoff objective, identical across roles.
3. **Role task:** the applicable template, filled with dynamic inputs and output requirements.

The complete prompt is sent as one user message, without a separate system message or provider tool definitions. Exact strings, optional sections, whitespace, and interpolation rules are in `prompt_templates.json`.

The **textual task description** is GLEE's literal observation `prompt` field, inserted unchanged under `## SERVER DESCRIPTION` for every model involved in that decision. It describes the game, role, payoff parameters, horizon, current situation, and server action instructions. The included descriptions cover all three game families and their observed action modes. They are observed examples; the historical server-side description generator is not included.

The orchestrator generates three state-specific questions. Each worker receives one question and the complete shared context, then proposes a complete action. The aggregator receives anonymous actions and rationales in a deterministic cyclic order and may choose a candidate or construct another action. Direct inference uses the same shared context and chooses an action in one call. Output field names, bounds, choices, counteroffer requirements, and split totals depend on the observation.

## Examples and scope

`example_prompts.json` contains exactly four complete prompt strings in `messages[0].content`, with no unfilled placeholders. The orchestrator, worker, and aggregator examples share one Negotiation observation with offer history. The direct example uses Persuasion; Bargaining descriptions and output formats are in the template file. The worker example represents the first of three parallel workers using the same template.

Game identifiers and opponent descriptors are sanitized. Questions and candidate actions are frozen recorded intermediate outputs, not fixed instructions. Schema title metadata is omitted; field descriptions and constraints are retained.

The template file describes dynamic construction, and the examples show the result. For a new request, supply its literal server description, visible context, calculated facts, legal-action constraints, and preceding stage outputs. This folder documents prompts and does not implement the game adapter, calculators, model clients, or live agent.
