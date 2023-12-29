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

Remix has some util functions to handle the file

```
import {
	unstable_createMemoryUploadHandler as createMemoryUploadHandler,
	unstable_parseMultipartFormData as parseMultipartFormData,
} from '@remix-run/node'
export const action = async ({ request }: ActionArgs) => {
	const uploadHandler = createMemoryUploadHandler({
		maxPartSize: 1024 * 1024 * 5, // 5 MB
	})
	const formData = await parseMultipartFormData(request, uploadHandler)

	const file = formData.get('avatar')

	// file is a "File" (https://mdn.io/File) polyfilled for node
	// ... etc
}
```

## Complex structure

👉 Array
```
<form>
	<input type="text" name="todo" value="Buy milk" />
	<input type="text" name="todo" value="Buy eggs" />
	<input type="text" name="todo" value="Wash dishes" />
</form>
```

```
const formData = new FormData(form)
formData.getAll('todo') // ["Buy milk", "Buy eggs", "Wash dishes"]
```
👉 Object

Since form doesn't support object, we could use Conform 😅 [See](https://conform.guide/complex-structures)

## Honeypot
Usually bots will try to find and submit any available forms 😅

Luckily, we can use Honeypot to prevent bots. The idea is pretty simple, we render an input that is visually hidden to human. If a bot fills in that input, we prevent the form from being submitted.

Another useful thing you can do along with the honeypot field is to add a field that will allow you to determine when the form was generated. So if the form is submitted too quickly, you can be pretty confident that it's a bot. This is more tricky than it sounds because bots could easily change the value of that field, so you do need to encrypt the value. But if you manage that, it's a great way to catch bots.

See more: [Remix utils HoneyPot](https://github.com/sergiodxa/remix-utils)

## Cross-site Request Forgery

This is a type of web security vulnerability where an attacker tricks a victim into performing actions they did not intent to do on a web application where they're authenticated.

## Rate limiting

Rate limiting helps to control the flow of incoming requests, ensuring that our server isn't overwhelmed. It does this by limiting the number of requests a user can make in a specific window of time. In the context of a web application, especially one that uses forms, rate limiting can be crucial.