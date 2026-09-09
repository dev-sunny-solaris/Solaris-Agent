# DatePicker

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/date-picker.blade.php`
- JS: `resources/js/solaris/form/date-picker.js`

`<x-core::date-picker>` supports `mode="date|datetime|time"`, `range`, `inline`, `lazy`, and `ignore`.
Range is applied only outside time mode; inline is likewise ignored for time.

`get()` returns a `Date`, a `Date[]` for range, or `null`. `toString()` is the payload representation:
date `Y-m-d`, datetime ISO, and time `H:i` (optionally seconds through runtime config). `set()` accepts a
string or Date-compatible value. Supported events are `change`, `open`, `close`, `ready`, and
`day-create`. Use lazy `init(config)` to override flatpickr configuration.
