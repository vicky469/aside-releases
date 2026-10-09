# Agents and Scripts

Aside supports local agents and reusable scripts in desktop Obsidian with a filesystem-backed vault. It can invoke Codex, Claude Code, Cursor, Gemini, and DeepSeek through local CLIs. Aside bundles none of those CLIs and has no agent service of its own. The `@deepseek` mention uses your configured OpenCode CLI and the model selected there, so the label does not guarantee that the active model is DeepSeek.

Scripts are an optional advanced capability. Scripts defaults on when an agent is available; a saved on/off choice takes precedence. See [Optional Advanced Capabilities](README.md#optional-advanced-capabilities) for how existing script usage affects the default. Ordinary agent replies do not require Scripts.

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

Ordinary `@codex` requests and agent-backed Codex script runs use the shared runtime. This changes process and connection reuse only: the global Default, the script card’s agent selection, independent reply cards, and local-only script behavior remain the same.

## Ask an Agent

Install and sign in to the local CLI you want to use first. In a side note, type `@codex`, `@claude`, `@cursor`, `@gemini`, or `@deepseek` with your request, then save the note. The CLI uses its configured account and model when Aside does not override them, but Aside supplies non-interactive execution and permission-affecting flags. Normal interactive approval behavior is not guaranteed.

Turn on **Settings → Aside → Sidebar → Enable agents** to show agent filters, suggestions, and actions. Turning it off hides agent controls; existing replies remain normal entries in the All view. Scripts have their own enable toggle.

## Enable Scripts

To create or run trusted local scripts, turn on **Settings → Aside → Scripts (advanced) → Enable scripts**. Then choose the default local agent in the same section for script creation, updates, and PDF conversion.

Turning Scripts off blocks new slash and script command execution plus script-oriented Generate actions. It does not delete registered scripts, history, or saved replies. Disabled script-like text remains ordinary note or agent text.

The Scripts setting is an experience and execution control, not a sandbox or security boundary. The security and privacy warnings above still apply whenever you run scripts or agents.

## Create or Update a Script

- Choose the default local agent used for script work under **Settings → Aside → Scripts (advanced)**.
- Use `/create-script <request>` in a side note. On the first valid request, Aside automatically creates `🛠️ scripts/` if the folder is missing, then asks the selected agent to create the script.
- Use `/update-script /script-name <request>` to ask the selected agent to change an existing script.
- Choose **Agent** at the bottom of the card. **Default** uses the agent selected in Settings; the dropdown selects one override for this request. The same control is available while editing a saved script command.
- Open a PDF and use `/pdf-to-markdown` by itself to ask the default agent to create a sibling Markdown file.

The dropdown stores its choice with the comment without inserting `@agent` text. Script commands no longer open a follow-up agent picker. Ordinary `@agent` mentions in non-script comments still work.

If creating or updating a script needs a decision, the agent asks and stops with **Waiting for reply**. Reply in the same thread to continue with the original request and selected agent.

## Run a Script

1. Open the Markdown note the script should process.
2. Add or reply to an Aside comment.
3. Type `/` and choose the script, or enter its command directly, such as `/clean-citations`.
4. For an agent-backed script, leave **Agent** on **Default** or select one agent in the card’s bottom dropdown.
5. Save the comment. Aside runs the script and appends its output to the thread.

Use one vault script per comment. Instructions follow the command, for example `/md-to-mindmap focus on causal relationships`. Local-only scripts have no agent dropdown and stay local. Agent reply cards show the agent identity rather than the script name.

Older saved comments without dropdown metadata retain their leading `@agent` selector behavior, including historical multi-agent runs. When editing one, the dropdown starts with its first legacy agent (or Default); saving stores that single selection. Existing text remains readable and is not rewritten. New draft cards use the dropdown.

Use **Generate** on a local script reply to run the latest version against the current note again. For an agent-backed result, per-agent Generate reuses that card’s stored prompt and original agent without rerunning script preparation or resolving Default again. For a failed preparation, Generate starts a fresh preparation using current invocation context and source, preserving the saved dropdown selection (or resolving Default again).

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

## Migrate Legacy Agent Scripts

The migration utility is repository maintenance code, not shipped plugin behavior, and it never runs inside Obsidian. Run it from an Aside source checkout with an explicit filesystem-backed vault path. Start with the dry-run, review every entry, and use `--apply` only after the proposed changes are correct:

```sh
node scripts/migrate-agent-vault-scripts.mjs --vault "/path/to/vault" --dry-run
node scripts/migrate-agent-vault-scripts.mjs --vault "/path/to/vault" --apply
```

The utility scans only direct eligible files in the vault's `🛠️ scripts/` folder and does not recurse. It recognizes these deliberately bounded legacy command shapes:

- Codex: `codex exec`
- Claude Code: `claude -p` or `claude --print`
- Cursor: `agent -p`
- Gemini: `gemini -p` or `gemini --prompt`
- OpenCode: `opencode run`

Recognition is conservative: refusal is safer than a speculative rewrite, so the analyzer refuses uncertain control flow, dynamic commands, ambiguous prompts, unsupported side effects, response-dependent work, and collisions; it does not guess. A supported rewrite keeps task-specific prompt preparation, removes provider and process plumbing, adds the Aside agent-script contract, and normalizes a `.js` or `.cjs` script to `.mjs` without changing its slash-command name.

Both `--dry-run` and `--apply` runtime-validate every convertible rendered module. Validation spawns Node, imports the rendered module and its dependencies, and calls `prepare` with a validation request, using the vault root as the working directory and the inherited environment. Dry-run means no migration transaction writes; it is not side-effect-free and does execute trusted module code. Run only trusted scripts and dependencies because they can read, change, or transmit anything the local account can access.

The report groups scripts as `changed`, `unchanged`, `skipped`, and `failed`. An analyzer refusal is `skipped`; an unsupported or ambiguous paired test, destination collision, or validation, planning, or apply problem is `failed`. Each file set is isolated, so unrelated valid file sets may still change when another one fails. Any failed entry or warning makes the command exit with code 1, even when some changed outputs were installed. Inspect `changed`, `failed`, `warnings`, and any `recoveryPath`, then rerun a dry-run before proceeding.

With `--apply`, each script and its paired tests use transactional per-file-set replacement with no-clobber staged and quarantine moves. A pre-commit failure attempts a complete rollback before commit. An incomplete rollback preserves and reports recovery evidence, including a `recoveryPath` or durability uncertainty when available. The files in a set are moved separately, not swapped as one indivisible filesystem operation.

Paired test handling covers both test and spec companions: it renames them with the script and updates imports that resolve to the old script filename. Specifically, matching direct `tests/<script>.test.*` and `tests/<script>.spec.*` files are renamed to `.mjs`. An unsupported or ambiguous paired test fails with the whole script file set unchanged. CommonJS paired tests and `.js` tests without positive static ESM syntax are reported as failed and require manual migration.

After applying, run the migrated paired tests and inspect the vault diff. The operation is idempotent: a second dry-run reports zero pending changes for successfully migrated scripts. Handle refused scripts manually or leave them unchanged; the utility intentionally supports only the documented shapes.

Treat both the legacy files and generated modules as trusted local code. Sensitive local content read by a script can enter its prepared prompt and reach the selected CLI and model provider. Review the generated prompt construction and the security guidance above before running a migrated script.

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

Aside owns the runtime, provider selection, persistence, and reply cards. A vault script only prepares task-specific prompt material: there is no vault-local base, sidecar, or provider launcher. Aside calls `prepare` once per invocation and sends the prepared prompt to the selected agent. Historical multi-agent invocations reuse that prompt for each stored agent.

Before registration and launch, Aside uses the source hash and contract inspection to verify the exact script snapshot and quarantine a malformed or changed agent script from runnable commands. It remains available as a unique authoring target. Repair it with `/update-script /script-name repair it` to trigger inspection again.

### Custom Context compatibility

Custom Context sources currently run local scripts only. The Context script picker excludes agent-backed scripts; if an existing source is converted to an agent-backed script, select a local script for that source. Run agent-backed scripts as `/script-name` in an Aside thread, where the card’s agent dropdown supports Default or one explicit agent.

## Local vault builds

`npm run build` validates the production build, then automatically runs `dev:install-all`. The installer discovers registered Obsidian vaults plus vaults under `~/Obsidian`, updates only existing Aside installations, verifies all three artifacts byte for byte, and reloads open vaults. Vault settings and plugin data are preserved. Machines without local Aside vaults skip installation.

Use `npm run dev:install-all` to repeat the sync, or `npm run dev:install-built -- --vault "/path/to/vault"` for an explicit installation. `dev:build-reload` is an alias for the full build. The development watcher remains scoped to its configured vault; run the full build after a change to update every vault.
