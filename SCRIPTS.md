# Agents and Scripts

Aside supports local agents and reusable scripts in desktop Obsidian with a filesystem-backed vault. It can invoke Codex, Claude Code, Cursor, Gemini, and DeepSeek through local CLIs. Aside bundles none of those CLIs and has no agent service of its own. The `@deepseek` mention uses your configured OpenCode CLI and the model selected there, so the label does not guarantee that the active model is DeepSeek.

Scripts are an optional advanced capability. Scripts defaults on when an agent is available; a saved on/off choice takes precedence. See [Optional agents and scripts](README.md#optional-agents-and-scripts) for how existing script usage affects the default. Ordinary agent replies do not require Scripts.

## Security

Vault scripts are **trusted, unsandboxed** code: they are **not sandboxed**. They run in local Node with your **local account permissions** and inherited environment. Aside launches Node without a shell and uses the vault root as its working directory. For a local-only script, Aside passes the absolute path of the current Markdown note as the script's only automatic argument. Output is bounded, and each run times out after 60 seconds.

These controls do not prevent a script from reading, changing, or sending anything available to your local account. Review every script and run only code you wrote or trust. Do not put credentials in a script or side note.

When Scripts is enabled, registry contract inspection imports each agent-backed module and its dependencies during startup, registration, and edit refresh, before any slash command runs. Inspection checks for a callable `prepare`; it does not call `prepare`. Top-level code in the module or its dependencies therefore runs during inspection. Review and trust that code before placing it in `🛠️ scripts/` or enabling Scripts.

### Agent Access and Privacy

Agent prompts include the saved side-note request, relevant source-note context, the thread transcript, the selected text when present, and paths needed for the task. The local CLI and its model provider may receive them. In particular, the note, thread, and selection may reach selected local CLIs and their providers. Agent runs may read or change vault files and use network-capable tools or external model providers within the tools, sandbox, and configuration supplied for that provider.

Aside starts agent transports headlessly with non-interactive, permission-affecting flags and protocol settings:

- The Codex app-server is a resident transport when compatible. It sets `approvalPolicy` to `never`, uses the `workspace-write` sandbox, and lists the run working directory plus, when distinct, the vault root as writable roots. Its one-shot fallback runs with `-s workspace-write`; Aside adds the vault root with `--add-dir` when needed.
- Claude Code receives `--allowedTools WebSearch,Bash,Read,Write,Edit,Glob,Grep`.
- Cursor receives `--force --trust --sandbox enabled`, using the workspace plus a vault `--add-dir` when needed.
- Gemini uses ACP as a resident transport when compatible and receives `--skip-trust --sandbox --approval-mode yolo` with the vault directory.
- `@deepseek` uses OpenCode ACP as a resident transport when compatible. Its one-shot fallback uses OpenCode `run --auto` with OpenCode's selected model.

Do not rely on normal interactive approval prompts. Review the CLI and provider settings, plus any sensitive note context, before invoking an agent.

Agent transports start lazily on first use. A compatible Codex app-server resident transport stays warm and idle between runs. Gemini ACP is resident when compatible, and OpenCode ACP is resident when compatible. Claude Code, Cursor, and incompatible versions remain one-shot.

Every invocation creates a fresh logical session, including retries and runs that share a resident transport; warm transport reuse is not persistent model memory. Aside supplies the saved note and thread context again for each invocation. An idle transport makes no inference calls and consumes no model tokens, but it retains small, non-zero device resources such as memory and process handles until Aside unloads or the transport exits or is restarted after a failure.

Ordinary `@codex` requests and agent-backed Codex script runs use the shared runtime. This changes process and connection reuse only: the global Default, explicit agent selectors, ordered multi-agent comparisons, independent reply cards, and local-only script behavior remain the same.

## Ask an Agent

Install and sign in to the local CLI you want to use first. In a side note, type `@codex`, `@claude`, `@cursor`, `@gemini`, or `@deepseek` with your request, then save the note. The CLI uses its configured account and model when Aside does not override them, but Aside supplies non-interactive execution and permission-affecting flags. Normal interactive approval behavior is not guaranteed.

Turn on **Settings → Aside → Sidebar → Enable agents** to show agent filters, suggestions, and actions. Turning it off hides agent controls; existing replies remain normal entries in the All view. Scripts have their own enable toggle.

## Enable Scripts

To create or run trusted local scripts, turn on **Settings → Aside → Scripts (advanced) → Enable scripts**. Then choose the default local agent in the same section for script creation, updates, and PDF conversion.

Turning Scripts off blocks new slash and script command execution plus script-oriented Generate actions. It does not delete registered scripts, history, or saved replies. Disabled script-like text remains ordinary note or agent text.

The Scripts setting is an experience and execution control, not a sandbox or security boundary. The security and privacy warnings above still apply whenever you run scripts or agents.

## Create or Update a Script

- Choose the default local agent used for script work under **Settings → Aside → Scripts (advanced)**.
- Use `/create-script [@agent] <request>` in a side note. On the first valid request, Aside automatically creates `🛠️ scripts/` if the folder is missing, then asks the selected agent to create the script.
- Use `/update-script /target [@agent] <request>` to ask the selected agent to change an existing script.
- Open a PDF and use `/pdf-to-markdown` by itself to ask the default agent to create a sibling Markdown file.

Without an explicit override, the familiar forms remain `/create-script <request>` and `/update-script /script-name <request>`.

`/create-script` and `/update-script /target` accept zero or one leading explicit agent. Zero uses the existing global Default. More than one distinct leading agent is rejected before any vault mutation. Later agent mentions in the request stay prose. Local scripts, `/create-script`, and `/update-script` do not automatically open the agent picker.

## Run a Script

1. Open the Markdown note the script should process.
2. Add or reply to an Aside comment.
3. Type `/` and choose the script, or enter its command directly, such as `/clean-citations`.
4. Save the comment. Aside runs the script and appends its output to the thread.

Use one vault script per comment. Agent-backed scripts accept supported agents only in the selector slot immediately after the command:

```text
/script-name @agent1 @agent2 [instruction ...]
/md-to-mindmap
/md-to-mindmap @codex
/md-to-mindmap @codex @gemini focus on causal relationships
```

Using zero explicit agents selects the existing global Default. Aside deduplicates supported selectors into an ordered, unique list, then creates a separate reply card for each agent in first typed order. Each card shows the agent identity, never the script name. Agent mentions later in the instruction are ordinary instruction text, not selectors.

Choosing an agent-backed script from slash suggestions automatically opens the agent picker. Select more agents to keep building the comparison. Pressing Escape or beginning instruction typing with zero selected agents closes the picker and means Default. Choosing a local script does not open the picker.

Local-only scripts ignore command-position agent selectors and stay local. Use **Generate** on a local script reply to run the latest version against the current note again. For an agent-backed result, per-agent Generate reuses that card's stored prompt and original agent without rerunning script preparation or resolving Default again. For a failed preparation, Generate starts a fresh preparation using current invocation context and source.

## Supported Scripts

- Scripts must be direct child files of `🛠️ scripts/`; nested folders are ignored.
- Supported extensions are `.mjs`, `.js`, and `.cjs`.
- Filenames cannot contain spaces. The filename without its extension becomes the command: `clean-citations.mjs` becomes `/clean-citations`.
- Hidden files and filename stems ending in `.test` or `.spec` are ignored.
- Command names are matched case-insensitively. Duplicate names are not runnable.
- Aside's built-in names are reserved and not runnable as scripts, including `todo`, `codex`, `claude`, `cursor`, `gemini`, `deepseek`, `create-script`, `update-script`, and `pdf-to-markdown`.

Aside keeps the live script registry current when eligible files are created, renamed, or deleted.

## Scripts Catalog

The **🛠️ Scripts** control beside the sidebar pin opens **Run once** and **Scheduled**. **Open catalog** opens `🛠️ scripts/Scripts.md`, whose frontmatter declares a shared timezone and whose embedded `Scripts.base` exposes script definitions and Last run. See [Scripts catalog and scheduling](docs/scripts.md) for the properties, migration behavior, and schedule approval workflow.

Local scripts declare `return_type: void` or `return_type: data` in their catalog record. For exported functions, set `entrypoint: run` and export `run({vaultPath, targetPath})` or a default function; existing scripts default to `entrypoint: cli`. Return nothing for `void`, return JSON-compatible data for `data`, and throw an Error for failure. Top-level boolean returns are invalid. A successful `void` run removes its result card while retaining its definition, schedule, and Last run, except index file batches retain a per-file success status. See [Scripts](docs/scripts.md) for multi-file Run once selection. Legacy command-line scripts remain supported.

## Agent-Backed Script Contract

Agent-backed scripts must use `.mjs` and the exact declaration `// aside-script: agent`. The declaration may follow an optional UTF-8 BOM, an optional shebang, and leading whitespace. The first 4 KiB boundary is measured after the optional BOM and shebang are removed; leading whitespace after them counts toward the boundary. The declaration must appear before executable code and be fully within the first 4 KiB. Repeated or unsupported declarations are invalid. Later marker-looking source text is ordinary content, not capability metadata.

Export one named `prepare(request)` function. It may be synchronous or asynchronous and returns an object containing the prepared agent prompt:

```js
// aside-script: agent

export async function prepare(request) {
    return {
        prompt: "Create a mind map from this Markdown:\n\n" + request.note.content,
    };
}
```

Aside supplies this closed Version 1 request shape:

```ts
interface AgentScriptRequestV1 {
    version: 1;
    script: { name: string; path: string };
    note: { path: string; content: string };
    instruction: string;
    thread: { text: string };
    selection?: { text: string };
}
```

The only accepted response shape is:

```ts
interface AgentScriptResponseV1 {
    prompt: string;
}
```

Unknown request or response fields are rejected. The serialized request must be at most 1 MiB, the verified source snapshot must be at most 1 MiB, and `prompt` must be nonblank and at most 64 KiB in UTF-8. Preparation times out after 60 seconds.

Aside owns the runtime, provider selection, persistence, and reply cards. A vault script only prepares task-specific prompt material: there is no vault-local base, sidecar, or provider launcher. Aside calls `prepare` once per invocation and sends the same prompt to all selected agents.

Before registration and launch, Aside uses the source hash and contract inspection to verify the exact script snapshot and quarantine a malformed or changed agent script from runnable commands. It remains available as a unique authoring target. Repair it with `/update-script /script-name repair it` to trigger inspection again.

### Custom Context compatibility

Custom Context sources currently run local scripts only. The Context script picker excludes agent-backed scripts; if an existing source is converted to an agent-backed script, select a local script for that source. Run agent-backed scripts as `/script-name` in an Aside thread, where Default-agent selection and explicit multi-agent comparisons are supported.
