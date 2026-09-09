# SolarUI Pages Router

Pages are complete screens that coordinate reusable SolarUI Components and page-specific behavior.
They are not registered with `SolarUI.register()`.

Resolve every package owner and source-relative path through
[Package registry](../../package-registry.md) before opening implementation files.

## Read only what the task needs

Choose the row before opening a detail guide. Do not load sibling detail files unless the Page
actually uses that feature.

| Task | Required Page guides | Required Component guides |
|---|---|---|
| Bespoke Page unrelated to List/Edit | [SolarPage](solar-page.md) | Only Components used by the view |
| Simple list without a form modal | [SolarPage](solar-page.md), [SolarListPage](solar-list-page.md) | [Table](../components/table.md) |
| Simple list CRUD with a modal form | [SolarPage](solar-page.md), [SolarListPage](solar-list-page.md) | [Table](../components/table.md), [ModalForm](../components/modal-form.md) |
| Custom list behavior or row actions | [SolarPage](solar-page.md), [SolarListPage](solar-list-page.md) | [Table](../components/table.md); [ModalForm](../components/modal-form.md) only when used |
| Inline-editable list | [SolarPage](solar-page.md), [SolarListPage](solar-list-page.md) | [Table](../components/table.md), [DataGrid](../components/datagrid.md) |
| Standard create/edit Page | [SolarPage](solar-page.md), [SolarEditPage](solar-edit-page.md) | [Form](../components/form/form.md), then only Components used by the view |
| Create/edit Page with list details | [SolarPage](solar-page.md), [SolarListPage](solar-list-page.md), [SolarEditPage](solar-edit-page.md), [SolarListDetail](solar-list-detail.md) | [Form](../components/form/form.md), [Table](../components/table.md); [ModalForm](../components/modal-form.md) only when used |

If scope changes during implementation—for example, a normal Table becomes `mode="grid"`—load the
newly required guide before editing code.

## Context decision

Read [Context](../../context.md) first. In brief:

- A **package** owns reusable source. Edit package files and test them through a sandbox.
- A **sandbox** is a throwaway consumer Laravel app managed by Solaris-Kit. Never edit its mirrored
  `vendor/` files.
- A **project** is a consumer Laravel app. Edit only app-owned files; installed packages are
  read-only.

Sandbox-owned and project-owned Page assets follow the same project-level resolution rules.

## Page family

| Base | Use it for |
|---|---|
| `SolarPage` | Bespoke complete screens outside standard list/edit flows |
| `SolarListPage` | Screens with one primary list Table |
| `SolarEditPage` | Standard create/edit screens |
| `SolarListDetail` | List-detail behavior inside its parent Page flow |
| `SolarModalPage` | Reserved; do not select until its dedicated guide is documented |

Never select a base by filename alone. Select it from the screen's interaction contract.
