# SolarModalPage

Owner: Core

Source-relative paths:

- JS: `resources/js/solaris/solar/solar-modal-page.js`

SolarModalPage coordinates an existing ModalForm as a Page-level controller. Pass either its ModalForm
instance or the base modal ID; string lookup first resolves `<id>_ModalForm`.

```js
import SolarModalPage from '@core-js/solaris/solar/solar-modal-page'

export default class StatusModalPage extends SolarModalPage {
	/**
	 * @param {import('@core-js/solaris/form/form').default} form
	 * @returns {void}
	 */
	onNew(form) {
		form.set('status_id', null, true)
	}
}
```

Construction resolves `this.modalForm`, then exposes `this.modal` and `this.form`, and finally calls
`init()`. Public `new()` and `edit(data)` delegate to ModalForm and invoke `onNew(form)` or
`onEdit(form, data)`. This is not a globally auto-initialized Component; another Page/controller must
construct it after SolarUI has created the target ModalForm.
