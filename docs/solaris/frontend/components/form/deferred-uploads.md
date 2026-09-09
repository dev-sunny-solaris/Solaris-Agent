# Deferred-upload Family Router

Owner: Core

Deferred upload means the record request completes first, then queued media receives the resulting ID.
Read [Form](form.md), then choose:

| Requirement | Guide |
|---|---|
| Crop and upload one avatar/profile image | [ProfilePicture](profile-picture.md) |
| Queue one or more arbitrary files | [Dropzone](dropzone.md) |

Register with Form rather than manually uploading in `success` when the upload depends on the newly
created record. Deferred upload failures are reported but do not roll back the successful record request.
