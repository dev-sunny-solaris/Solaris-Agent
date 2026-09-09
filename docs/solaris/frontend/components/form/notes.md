# Notes

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/notes.blade.php`
- JS: `resources/js/solaris/notes/notes.js`

Notes is a Quill editor managed as a Form component. Props include `id`/`bind`, `lazy`, `ignore`, record
and model context for media, `column`, and `pdf_filename`. Slot HTML initializes content.

`get()` returns Quill Delta, `getJson()` returns serialized Delta, and `getRaw()` returns HTML. Form
serializes Notes with `getRaw()`. `set()` accepts Delta, JSON Delta, HTML, or empty input; `reset()`,
`isEmpty()`, `disabled()`, and `enabled()` are available. Embedded image/video uploads require model and
record context. PDF export uses the configured column and filename. No public component events are
declared.
