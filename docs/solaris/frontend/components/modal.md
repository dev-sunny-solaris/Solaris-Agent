# Modal

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/modal.blade.php`
- JS: `resources/js/solaris/modal.js`

Modal wraps Bootstrap Modal. Props configure `id`, static backdrop, fullscreen, size, centering,
animation, title, close control, and `ignore`. Default content is body; named `header` and `footer`
replace/add those regions. Route attributes with `header:`, `body:`, and `dialog:`.

Use `show()` and `hide()` and listen to `show`, `shown`, `hide`, or `hidden`. Named listeners are
replaceable/removable. Use ModalForm when the modal's primary responsibility is submitting a Form;
do not manually rebuild that composition.
