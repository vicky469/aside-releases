# Context and connections

## Thought Trail

Thought Trail brings vault relationships together through Wikilinks and Tags. Wikilinks follow references between ordinary Markdown notes; Tags show notes that share note or side-note tags. Canvas boards, attachments, and Excalidraw drawings (including drawings stored as `.md`) are excluded from Thought Trail, including their side-comment links.

## Context

Context provides **Files** followed by **URLs** for the current note or Canvas and its saved side comments. Saved comment bodies are always included in both lists. Short URL labels retain the full destination for opening and copying.

Files lists resolved vault links and embeds, regardless of folder: Canvas boards, Excalidraw drawings, PDFs, DOCX documents, images, and other attachments. Excalidraw drawings also appear when saved as `.excalidraw.md` or as ordinary `.md` files marked by Excalidraw. For Markdown sources, ordinary Markdown links belong in Thought Trail. For an open Canvas, Files also includes Markdown notes referenced by file cards or text-card links, because Canvas has no Thought Trail. Canvas URL cards and text-card URLs appear under URLs; unsaved board content is read from its current native view. Files are sorted alphabetically by type, then filename. Duplicate references appear once; click a filename to open it through Obsidian.

These sources are automatic. Link or embed a file in the note or a saved side comment to associate it; no **+** button or script configuration is needed. File detection uses Obsidian's cached metadata.

Previously saved custom Context sources remain available for notes sharing their `source_type`, or the Notes group when none is set. Their definitions use trusted registered code from `🛠️ scripts`, and generated output remains note-specific. Existing sources can still be edited or removed. Scripts run only when you choose Generate; opening Context or changing a note does not run them. Custom output renders as Markdown and may include links or Canvas embeds.
