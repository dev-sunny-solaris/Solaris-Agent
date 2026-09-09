# Text-input Family Router

Owner: Core

Read [Form](form.md) first when the input is Form-owned. Then read exactly one child guide:

| Requirement | Guide |
|---|---|
| Plain text, textarea, numeric/mask formatting | [Text and Mask](text-mask-input.md) |
| Debounced search box with search/reset action | [SearchInput](search-input.md) |
| Tags or mixed inline tags | [TagInput](tag-input.md) |
| Date, datetime, time, or range | [DatePicker](date-picker.md) |
| Password visibility or one-time code | [Password and OTP](password-otp.md) |

All children follow BaseInput conventions: `get()`, `set(value, isSilent)`, `reset()`, visibility,
enabled/disabled state, validation state, and named event listeners. Returned value types differ; never
assume every text-looking control serializes a string.

Field compatibility is narrower than standalone/Form ownership. Read [Field](field.md) before nesting a
specialized input inside `<x-core::field>`.
