# Professional Web Forms
Topic to cover
1. Validating user input and displaying error messages
2. Fixing Accessibility bugs
3. Using a schema to validate data
4. Uploading files
5. Managing form submissions of complex data structures
6. Protecting your forms with a Honeypot
7. Protecting your users with CSRF tokens
8. Protecting yourself with rate limiting

## Form validation
- HTML has built-in support for validation of form elements (eg: required, minlength, ...). Only client-side validation and it should be seen as a nice progressive enhancement for better UX.
```
<input required maxLength={100}>
```
See more [Client-side form validation](https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation)
- We need to do the validation also on the server (Remix action)
```

type ActionErrors = {
    formErrors: Array<string>,
    fieldErrors: {
        title: Array<string>
    }
}

function action({request}: DataFunctionArgs) {
    const formData = await request.formData()
    const title = formData.get('title)

    const errors: ActionErrors = {
        formErrors: [],
        fieldErrors: {
            title: []
        }
    }

    if (title.length > 5) {
        fieldErrors.title.push('Title shouldn't > 5')
    }

    const hasErrors = errors.formErrors.length || Object.values(errors.fieldErrors).some(errors => errors.length)

    if (hasErrors) {
        return json({status: 'error', errors}, {status: 400})
    }
}

// Client
function Page() {
    const actionData = useActionData<typeof action>()
    const fieldErrors = actionData?.status === "error" ? actionData.errors.fieldErrors : null

    // then we can display the error
    {fieldErrors?.title.length ? (
		<ul>{fieldErrors.title.map(e => (<li key={e}>{e}</li>))}</ul>
	) : null}
}
```
- Disable default validation
We want to disable the built-in browser errors but we don't want to disable them completely, just once our JavaScript has loaded to enhance the user experiment. 

```
function useHydrated() {
	const [hydrated, setHydrated] = useState(false)
	useEffect(() => setHydrated(true), [])
	return hydrated
}

<Form noValidate={hydrated}>
```

## Accessibility
```
- Screen reader
- Focus Management
- Color contrast
- Keyboard navigation
- ARIA
- Animations
- Language
- etc...
```

- Need to use ARIA to associate error messages to the fields
```
<form>
    <label for="name">Name</label>
    <input id="name" type="text" aria-invalid="true" aria-describeby="name-error"/>
    <div id="name-error">Name is required</div>
</form>
```
- If submit or reset button is rendered outside the form, we need to associate it with the form via `id`
```
const formId = 'note-editor'

<form id={formId}>...</form>
<button form={formId} type="reset">Reset</button>
<button form={formId} type="submit">Submit</button>
```
### Focus management
- We can add `autoFocus` to the title field, so users can start typing right away
```
<input autoFocus/>
```
- We can focus on the first input field which has an error
- If the entire form has an error? Well, we focus on the form itself

```

const formRef = useRef<HTMLFormElement>(null)

useEffect(() => {
    const formRef = formRef.current

    if (!formRef) return 
    if (actionData?.status !== 'error') return

    if(formEl.matches('[aria-invalid="true"]')) {
        formEl.focus()
    } else {
        const firstInvalidField = formEl.querySelector('[aria-invalid="true"]')

        if (firstInvalidField) {
            firstInvalidField.focus()
        }
    }
}, [actionData])

// tabIndex = -1 means this is not something user can tab around
// BUT we can programmatically focus on
<form tabIndex="-1">
```

We can abstract this into a hook
```
useFocusInvalid(formRef.current, hasErrors)
```

## Schema Validation
We can do both server and client validation with Zod + Conform

Check this doc: https://conform.guide/tutorial

## File upload

We use the `<input type="file"/>` element that enables interaction with the file system. This element creates an user interface that allows users to select one or multiple files from their system.

```
<form action="/upload" method="post" enctype="multipart/form-data">
	<label for="file-upload-input">Upload File</label>
	<input type="file" id="file-upload-input" name="file-upload" />
	<button type="submit">Upload File</button>
</form>

```

* `enctype` attribute: set to `multipart/form-data` to ensure the file data is sent correctly to the server
* `multiple` attribute: for multiple file selection