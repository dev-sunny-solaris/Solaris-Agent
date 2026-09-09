# StageButtons

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/stage-buttons.blade.php`
- JS: `resources/js/solaris/stage/stage-button.js`
- Stage JS: `resources/js/solaris/stage/stage.js`
- Dropdown stage JS: `resources/js/solaris/stage/dropdown-stage.js`

StageButtons represents workflow/status progression and is Form-serializable. Each stage provides
`id`, `name`, `color`, optional `confirm`, `disabled`, `next`, and optional dropdown `menu`. Props include
`initial`, current `value`, `mode` (`next` or `next-prev`), dropdown mode, `final`, `bind`, and `ignore`.

`get()` returns `{ id, name }` or `null`; Form sends the ID. `set(idOrName, isSilent)` activates a stage
or dropdown item. Events are `click`, `change`, and `change:after`. An async `click` listener can veto a
transition by returning a value other than `true`, `null`, or `undefined`. Use `setConfirm()` for runtime
confirmation and transition methods only when workflow rules require them.

`lazy` is emitted by Blade, but StageButtons currently does not use it as a constructor guard. Do not
assume lazy behavior without checking Core.
