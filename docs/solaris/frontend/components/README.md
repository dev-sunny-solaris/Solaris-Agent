# SolarUI Components Router

Components are reusable atomic UI behavior or reusable compositions of atomic elements. A Page may
coordinate multiple Components, but Component contracts do not belong in Page documentation.

Resolve every package owner and source-relative path through
[Package registry](../../package-registry.md) before opening implementation files.

## Read only the used Components

| UI used by the task | Guide |
|---|---|
| Server-backed or local Table | [Table](table.md) |
| Inline-editable Table | [Table](table.md), then [DataGrid](datagrid.md) |
| Form inside a Modal | [ModalForm](modal-form.md) |
| Form ownership, submission, validation, and serialization | [Form](form/form.md) |
| Text, masked, search, tag, or date/time input | [Text-input family](form/text-inputs.md) |
| Select, Lookup, radio, checkbox, or option group | [Option-input family](form/option-inputs.md) |
| Searchable dropdown or model reference | [Lookup](form/lookup.md) |
| Upload after the main Form request | [Deferred-upload family](form/deferred-uploads.md) |
| Rich text | [Notes](form/notes.md) or [DocumentEditor](form/document-editor.md) |
| Workflow/status steps | [StageButtons](form/stage-buttons.md) |

Coverage is intentionally incremental. Modal, SolarModalPage, Field, Button, Tab, Fieldset, Attachment,
and TableLookup require dedicated guides before agents treat them as documented contracts.

Do not load DataGrid for a normal Table. Do not load ModalForm for a list that has no modal form.
Read a family router first, then only the child guide selected by that router.
