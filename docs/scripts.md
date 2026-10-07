# Scripts

Enable **Scripts** in Aside settings, then select **🛠️ Scripts** beside the sidebar pin. **Run once** executes a script on demand; **Events** runs scripts when matching notes are created, modified, or renamed; **Scheduled** manages runs on the connected server. Opening a tab never runs a script.

In the index sidebar, use **Choose files** to search and select multiple vault files, then **Run on N files**. Each script processes the selected files one at a time through the configured local or remote runner. File selection overrides the catalog's default target for that batch. Each file has a queued, running, succeeded, or failed result; a failure does not stop the remaining files. **Retry** reruns only that failed file. Selection and results last for the current plugin session; scheduled runs continue to use their catalog targets.

## Event rules

Open **Events**, choose a script and **File created**, **File modified**, or **File renamed**, then optionally enter `source_type`, category, and folder filters. **Save rule** persists the rule; an enabled rule begins watching future events. **Pause** stops a rule, and **Edit** changes it. Event rules apply across the vault, regardless of which note is open in the sidebar. The script receives the event file as its target.

For YouTube transcript cleanup, select `clean-youtube-transcript`, choose **File created**, set `source_type` to `youtube`, leave **Enabled** checked, and save. Source type and category values are case-sensitive exact matches; a list property matches if any item equals the filter. Empty filters match all notes. Folder paths are vault-relative without a trailing slash and include subfolders.

Event detection runs while this vault is open in Obsidian and Scripts is enabled. Only Markdown notes are watched; script/catalog files, `Scripts.md`, and the derived Aside index are excluded. Existing notes are not replayed on startup. Creation rules allow 30 seconds for an importer to add matching properties, running each matching rule once for that creation. Rapid changes are debounced, runs are processed sequentially, and the script's writes to its target are suppressed to prevent loops. Avoid scripts that create or modify other notes matching the same rule: those notes produce their own events. Missed events are not replayed after reopening Obsidian. Each open device detects events independently, so enable automation on only one device when duplicate processing would matter.

Rules are separate catalog records, so adding a trigger does not change the script's manual or scheduled settings. The latest result for each rule and file, including successful void runs and errors, appears in Events for the current session (up to 100 rule/file pairs). **Last run** remains stored in the catalog. A failure does not prevent processing later files. Newly generated `Scripts.base` files include an Events view; existing customized Bases are preserved, and all rules can be edited from the Events sidebar.

## Catalog and timezone

**Open catalog** creates or opens `🛠️ scripts/Scripts.md`, which embeds `🛠️ scripts/Scripts.base`. Its frontmatter owns the shared scheduling timezone. Existing root-level catalog files move into `🛠️ scripts` when the catalog opens; customized contents are retained:

```yaml
---
timezone: Asia/Shanghai
---
```

The initial timezone comes from the device when the catalog is first created. It is then fixed until you edit it; travelling or opening the vault on another device does not change scheduling. A daily `09:00` means 09:00 in this timezone. Interval schedules use elapsed minutes. Daily times skipped by daylight saving do not run that day; repeated times run once.

The Base edits Markdown records in `🛠️ scripts/catalog/`. Registered local scripts receive a record with `return_type: data`, `run_once: true`, and no enabled schedule. Existing server jobs are imported when Scheduled opens; their IDs, targets, intervals, and enabled state are retained. Apply schedules attaches imported jobs to their catalog records. Existing agent-backed scripts retain their separate prompt-preparation contract.

| Property | Meaning |
| --- | --- |
| `script_id` | Stable identity; do not duplicate it. |
| `name`, `description` | Script identity and purpose. |
| `script` | Registered executable path under `🛠️ scripts/`. |
| `return_type` | `void` or `data`. |
| `entrypoint` | `cli` for existing command-line scripts; `run` for exported functions. Editable in All scripts. |
| `run_once` | Whether the script appears in Run once. |
| `source_type`, `category` | Sidebar visibility filters and event-rule conditions. Empty means all notes. They do not change execution targets. |
| `event` | `none` (default), `file-created`, `file-modified`, or `file-renamed`. |
| `event_enabled` | Whether this event rule is active; missing defaults to false. |
| `event_folder` | Optional folder filter for event rules, including descendants. |
| `target` | Default vault-relative note/folder path. `.` uses the current note for note-sidebar Run once, or the whole vault for scheduled runs. Index batches use the explicitly selected files. |
| `schedule` | `manual`, `interval`, or `daily`. |
| `interval_minutes` | Whole minutes between interval runs, from 1 to 525600. |
| `at` | Daily time, quoted as `"09:00"`. |
| `schedule_enabled` | Whether the configured schedule is active. Paused schedules stay visible. |
| `last_run` | Automatically recorded UTC start datetime, including attempts that fail. |
| `last_run_display` | Automatically formatted **Last run** table value in the shared timezone. |

The display property is maintained separately because native Base date formatting follows the viewing device. Neither Last run field should be edited manually. Before the first run both are blank. Changing the shared timezone updates the display without changing the stored instant.

Use the **All scripts** Base view to configure a currently hidden script or add its schedule. Then choose **Apply schedules** in Aside. This syncs the catalog and approves the current executable/target configuration. Later schedule edits sync to the server; changed executable code still invalidates its approval. A disconnected or paused sync connection cannot apply changes. Schedules continue remotely while Obsidian is closed.

## Return contract

For new local scripts, set `entrypoint: run` in the catalog record and export `run` or a default function:

```js
export async function run({ vaultPath, targetPath }) {
  if (!targetPath) throw new Error("A target is required.");
  return { targetPath, vaultPath, message: "Ready" };
}
```

Declare `return_type: data` for text, finite numbers, objects, arrays, or null. Returned data remains in a run card. Declare `return_type: void` for a function returning nothing; its card disappears after successful completion except in index batches, which retain a per-file success status. The catalog row, schedule controls, and Last run remain. A declared/actual return-type mismatch is an error.

Throw an `Error` when a script fails. A top-level `true` or `false` return is rejected; booleans inside structured data are allowed. Failed run cards remain available for inspection/retry. Local processes and scheduled jobs retain their existing time and output limits.

Existing records default to `entrypoint: cli`, which launches legacy command-line scripts directly. This preserves their main-entry guards and prevents double execution: the target is still `process.argv[2]`, and the working directory is the vault root. For `data`, stdout supplies the result; for `void`, successful completion has no retained output. A nonzero process exit reports failure. Bare boolean stdout is rejected as a success/failure signal.
