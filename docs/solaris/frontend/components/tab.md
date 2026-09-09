# Tab

Owner: Core

Source-relative paths:

- Blade: `resources/views/components/tab.blade.php`
- Nav Blade: `resources/views/components/tab-nav.blade.php`
- Content Blade: `resources/views/components/tab-content.blade.php`
- JS: `resources/js/solaris/tab.js`

Tab pairs `nav` and `content` slots. Every TabNav ID must match its TabContent target ID. Use nested Tabs
for grouped sections instead of manually toggling panes.

`getActive()` returns the active target ID. `setActive(id)`, `show(idOrIds)`, `hide(idOrIds)`,
`enable(idOrIds)`, and `disable(idOrIds)` coordinate nav and pane state. The `change` event receives the
activated target ID and Bootstrap event.
