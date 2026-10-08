
<p align="center">
  <img src="./assets/logo-readme.svg" alt="Aside logo" width="72">
</p>
<p align="center">
Aside
</p>
<p align="center">
  <a href="https://github.com/vicky469/aside-releases/releases/tag/2.0.112">
    <img src="https://img.shields.io/badge/release-2.0.112-22c55e?style=flat-square" alt="Latest release">
  </a>
</p>
<p align="center">
  <a href="https://obsidian.md">
    <img src="https://img.shields.io/badge/Obsidian-API-7c3aed?style=flat-square&logo=obsidian&logoColor=white" alt="Obsidian API">
  </a>
  <a href="https://www.typescriptlang.org/">
    <img src="https://img.shields.io/badge/TypeScript-language-3178c6?style=flat-square&logo=typescript&logoColor=white" alt="TypeScript">
  </a>
  <a href="https://codemirror.net/">
    <img src="https://img.shields.io/badge/CodeMirror-editor-0ea5e9?style=flat-square" alt="CodeMirror">
  </a>
  <a href="https://lezer.codemirror.net/">
    <img src="https://img.shields.io/badge/Lezer-parser-f59e0b?style=flat-square" alt="Lezer">
  </a>
</p>
<table>
  <tr>
    <td align="center" valign="top" width="50%">
      <strong>Side note index</strong><br>
      <img src="./assets/demo.gif" alt="Aside demo preview in Obsidian dark theme" width="100%">
    </td>
    <td align="center" valign="top" width="50%">
      <strong>Agent reply</strong><br>
      <img src="./assets/demo2.gif" alt="Aside demo preview in Obsidian light theme" width="100%">
    </td>
  </tr>
</table>
Aside is a tool for thought. Add side notes to your Obsidian files, connect ideas, and get optional help from local AI agents.

## Features

- **Comment beside your notes.** Add page notes to files, or attach comments to Markdown selections and Canvas cards.
- **Keep track of ideas.** Pin comments, add `#tags` and `[[wikilinks]]`, and mark follow-ups with `@todo`.
- **Find connections.** Browse comments in `🐰 Aside Index.md`, explore links and tags with Thought Trail, and see attachments and URLs in Context.
- **Ask an agent.** Mention `@codex`, `@claude`, `@cursor`, `@gemini`, or `@deepseek` for help on desktop.

See [Context and connections](docs/context.md) for more detail.

For durable storage and sync across devices, use Aside with [Obsidian Sync](https://obsidian.md/sync).

## How to Get Started

Community directory submission is pending. Until Aside is available there, use the manual installation instructions below.

1. Open **Settings → Community plugins** in Obsidian, find **Aside**, and install and enable it.
2. Open a file and choose **Add page note**, or select Markdown text or a Canvas card and choose **Add comment to selection**.
3. Write your comment and click **Add**.

Requires Obsidian **1.12.7** or later. To update, select **Check for updates** in Community plugins.

For manual installation, download `main.js`, `manifest.json`, and `styles.css` from the [same release](https://github.com/vicky469/aside-releases/releases/latest), copy them into your vault's `.obsidian/plugins/aside/` folder, and reload Obsidian. Preserve your existing plugin settings and data.

[Community listing](https://community.obsidian.md/plugins/aside) · [Release notes for 2.0.112](docs/releases/2.0.112.md)

## Workflow

1. Open a file and add a page note, or select Markdown text or a Canvas card and choose **Add comment to selection**.
2. Write your comment. Use `[[wikilinks]]` to connect notes, `#tags` to organize ideas, and `@todo` for follow-ups.
3. Click **Add**. Use **+** on a saved card to continue the thread with a reply. When editing an existing entry, click **Save**.

## Glossary

- **Thread** — One Aside discussion attached to a file, a text selection, or a Canvas card. It contains a first entry and any replies.
- **Entry** — One message in a thread. The first saved entry creates the thread; later entries are replies.
- **Page note** — A thread attached to a whole file. Page notes work in supported file views, including those provided by viewer plugins. The generated Aside Index is not a page-note target.
- **Anchored note** — A thread attached to a Markdown text selection or a Canvas card. Selecting the comment takes you back to its target.
- **Orphaned note** — An anchored thread whose original text or card can no longer be found. The conversation is preserved even when its anchor is missing.
- **🐰 Aside Index.md** — The generated vault-wide comment index. It helps you browse your comments; it is not their primary storage.
- **Thought Trail** — The relationship view that follows note links and connects files with shared note or side-note tags.

## Writing in Side Notes

| Action | How |
| --- | --- |
| Add a comment or reply | Click **Add**. |
| Save an edit | Click **Save**. |
| Reply to a thread | Click **+** on a saved card. |
| Ask a local agent | Mention `@codex`, `@claude`, `@cursor`, `@gemini`, or `@deepseek` in a new comment or reply, then click **Add**. |
| Link a note | Type `[[` to find or create a note. |
| Add a tag | Type `#` and choose or create a tag. |
| Reopen link or tag suggestions | Press `Tab` inside an unfinished `[[...` or `#...` token. |
| Mark a follow-up | Type `@todo`. |
| Keep a comment handy | Pin its card. |
| Add a line break | Press `Enter`. |
| Cancel an edit | Press `Esc`. |

## Commands

- **Aside: Add comment to selection** — Add an anchored comment to selected Markdown text or a Canvas card.

## Optional agents and scripts

Install and sign in to your preferred local agent CLI, then turn on **Settings → Aside → Enable agents**. Mention the agent in a new comment or reply and click **Add** to start a conversation.

`@deepseek` uses the model selected in your configured OpenCode CLI. Scripts let you run and schedule trusted local commands; only run scripts you trust.

See [Agents and scripts](SCRIPTS.md) for setup and [Scripts and scheduling](docs/scripts.md) for automation.

## Distribution

This repository contains compiled release assets and user documentation. Current development source is maintained separately in a private repository.

## Accounts, network, and file access

- Aside's comment features do not require an Aside account. Optional agents use your installed Codex, Claude Code, Cursor, Gemini, or OpenCode CLI and its configured model provider. These services may require an account or payment. Agent requests can send your request, relevant note content, thread, selection, and file paths to the selected provider to generate replies.
- On desktop, Aside launches installed CLIs and trusted scripts using your local account permissions. It uses temporary files outside the vault for agent execution. Agents and scripts may access other files and network services within their configured permissions; scripts are not sandboxed. See [Agent access and privacy](SCRIPTS.md#agent-access-and-privacy) before enabling them.
- The generated Aside Index includes a header image served by `ichef.bbci.co.uk` (BBC); displaying it can contact that host. Remote images and web pages you open can also contact their respective hosts.

## License

Aside is free for personal and internal workplace use under the [Aside Proprietary License](LICENSE). Your notes remain yours. Earlier MIT-licensed releases retain their permissions.
