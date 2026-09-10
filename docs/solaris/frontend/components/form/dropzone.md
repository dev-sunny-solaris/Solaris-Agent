# Dropzone

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/dropzone.blade.php`
- JS: `resources/js/solaris/dropzone.js`

Dropzone queues and chunk-uploads files. Props configure `chunk_size`, `max_file_size`, `multiple`,
`auto`, `url`, `param_name`, `table_name`, `record_id`, accepted extensions/MIME values, `lazy`, and
display size. Core clamps requested file size to its global maximum and checks storage quota.

Events: `addedfile`, `sending`, `success`, `error`, `removedfile`, and `queuecomplete`. With `auto=false`,
files remain queued; `hasPendingFiles()` detects them and `upload(config)` resolves boolean after the
whole queue completes. Runtime setters can update URL, record/model identity, accepted files, auto mode,
and multiplicity.

```js
this.form.registerDropzone(
	this.attachments,
	response => ({ recordId: response.data.id })
)
```

The resolver may return `{ url, recordId, tableName }`. Use `url` for a custom endpoint; use `recordId`
with the standard attachment endpoint. Do not also serialize Dropzone as an ordinary field.
