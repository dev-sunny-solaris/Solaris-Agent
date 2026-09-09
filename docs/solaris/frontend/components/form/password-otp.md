# Password and OTP

Owner: Core

Source-relative paths:

- Password Blade: `resources/views/components/password.blade.php`
- Password JS: `resources/js/solaris/form/password-input.js`
- OTP Blade: `resources/views/components/otp-input.blade.php`
- OTP JS: `resources/js/solaris/form/otp-input.js`

Password extends TextInput with a visibility toggle and can be used inside Field. It preserves normal
text-input value and events; visibility methods affect its wrapper.

OTP is a standalone global Component, not a Field plugin or normal Form-owned input. It supports numeric
or alphanumeric modes, configurable length, optional resend URL/method, `get`, `reset`, `focus`, display
and error APIs, resend configuration/timer, plus `change` and `resend` events. `change` fires when all
digits are filled. Submit its value explicitly through the owning flow when required.
