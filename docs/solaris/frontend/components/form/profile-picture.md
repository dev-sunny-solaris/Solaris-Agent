# ProfilePicture

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/profile-picture.blade.php`
- JS: `resources/js/solaris/profile-picture.js`

ProfilePicture displays an image, validates a selected PNG/JPEG, opens a cropper, converts/compresses to
WebP, uploads through Dropzone, and optionally opens an existing attachment in PhotoSwipe. Main props:
`id`, shape/size, `source`, `upload_to`, `param_name`, `thumbnail`, maximum size, preview dimensions, and
`view_only`.

Events: `change` receives the raw file, `cropped` follows crop preparation, and `success` receives upload
and local preview data. `hasPendingCrop()` identifies deferred work. Direct `upload(url)` is valid when
the URL already exists.

For create forms:

```js
this.form.registerProfilePicture(
	this.profilePicture,
	response => `${BASE_URL}/accounts/${response.data.id}/avatar`
)
```

The URL resolver must return a URL. Form calls `upload(url, true)` only when a crop is pending.
