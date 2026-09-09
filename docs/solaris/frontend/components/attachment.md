# Attachment

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/attachment.blade.php`
- JS: `resources/js/solaris/attachment.js`

Attachment is an existing-record file manager composed from an internally owned Dropzone, Table, and
preview Modal. Supply `table_name` and `record_id`; legacy `relation="table:id"` remains accepted but is
deprecated.

Uploads refresh the list when their queue completes. The list provides preview, download, and deletion.
Use public `refresh()` and `download(id)` rather than resolving ignored internal children. Attachment is
not the deferred create-record uploader; use Form-registered Dropzone when no parent record exists yet.
