# DocumentEditor

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/document-editor.blade.php`
- JS: `resources/js/solaris/document-editor.js`

DocumentEditor is the CKEditor-based full document editor. Use it for document-oriented formatting,
tables, page breaks, word count, fullscreen, and PDF export; use [Notes](notes.md) for Quill/Delta-based
notes. Props supply `id`/`bind`, `ignore`, record/model/column context, and `pdf_filename`. Slot HTML is
initial editor data.

Initialization is asynchronous. `get()`/`getRaw()` return HTML, `set()` replaces HTML, and `reset()`,
`isEmpty()`, read-only toggles, context setters, and `exportPdf()` are available. Embedded uploads use
the Core attachment endpoint and need model/record context. No public events are declared.

Although Blade exposes `lazy`, the current JS constructor initializes immediately; do not rely on lazy
DocumentEditor initialization without verifying/fixing Core behavior.
