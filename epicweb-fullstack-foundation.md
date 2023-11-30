## 1. Styling
- Assets import: Remix caches static asset for 1y (depending on our server). If we update the assets, we also need to update the asset name (cache key), so browser can download the asset again. Another better solution is to import the asset and Remix will take care of getting it into the `public` directory and give us the URL to it. As part of this process, Remix will fingerprint the file. 
```
import faviconAssetUrl from './assets/favicon.svg'
```
- Remix has built-in supports for Tailwind + PostCSS
## 2. Data loading
- Throw Response 🤯 It allows you to easily exit the code flow and send a response right away. Remix will handle catching error for us
```
throw new Response('Page not found', {status: 400})
```
We can even put this as a util function
```
function assertDefined<Value>(value: Value | null | undefined): asserts value is Value {
    if (value === undefined || value === null) {
		throw new Response('Not found', { status: 404 })
	}
}
```
## 3. Data mutation
- Broswer form only supports FETCH and POST. Other actions will be fallback to GET.
- Form with GET action will seriallize inputs into a "query string"
- Form with POST action will submit the form body as a "payload". The browser will encode it as `application/x-www-form-urlencoded`
- Client-side focused applications (Eg: handling delete button). We will run into different edge cases
	- What if the user clicks the button twice?
	- What if the user clicks the button while the request is still in flight?
	- What if the user clicks the button while the request is still in flight, and then navigates away from the page?
	- What if the request fails?
	- What if the user clicks the button twice, but the second response returns first?
- Remix will handle that for us automatically
- If there are many buttons in the form, we can add `name="intent"` to the button
```js
<Form>
	<Button name="intent" value="delete">Delete</Button>
	<Button name="intent" value="edit">Edit</Button>
</Form>

// server
const formData = await request.formData()
const intent = formData.get("intent")
// check intent and do action
```

## 4. Scripting
**Javascript**
We need Javascript for: 
- Navigations
- Pending UI
- Accessibility
- Prefetching

To solve the "full page reload" proble, we have to take a lot of the browser's responsibilities upon ourselves. We end up using `event.preventDefault()` on a link clicks and form submissions. Unfortunately, the browser does a lot of work for us that we have to re-implement ourselves. 

- Scroll restoration: the browser is no longer responsible for navigations as we use Remix. Luckily, Remix has a `ScrollRestoration` component
- In Remix, we build 3 "bundles": node server, Remix server, client
- Server can have access to the `process.env`, however, the client doesn't. We need to send those variables (not private ones) to the client via loader and attach them to the window object.

**Prefetching** 
- Remix Link can prefetch resources when users hover a link 
- Prefetch mode: none, intent, render, viewport. Check the [docs](https://remix.run/docs/en/main/components/link#prefetch)

**Pending UI**
- We can know if the user is submitting a form, navigate to a new page by using `useNavigation` + `useFormAction`

```ts
	const navigation = useNavigation()
	const formAction = useFormAction()
	const isSubmitting = navigation.state != "idle" && navigation.formMethod === "POST" && navigation.formAction === formAction
```

We can extract this logic into a util function

```ts
export function useIsSubmitting({
	formAction,
	formMethod = 'POST',
}: {
	formAction?: string
	formMethod?: 'POST' | 'GET' | 'PUT' | 'PATCH' | 'DELETE'
} = {}) {
	const contextualFormAction = useFormAction()
	const navigation = useNavigation()
	return (
		navigation.state === 'submitting' &&
		navigation.formAction === (formAction ?? contextualFormAction) &&
		navigation.formMethod === formMethod
	)
}
```

## 5. Search Engine Optimization
- Remix metadata doesn't get merged automatically. It always uses the meta of a leaf route (lowest route)
- Remix Links get merged
- Meta function will run even when the loader throws error. In that case, `data` returned from the loader might be undefined.
- We can also get loader data from other routes
```
export const meta: MetaFunction<typeof loader, {'root', typeof rootLoader}> = ({matches}) => {
	const rootLoaderData = matches.find(m => m.id === "root")
}
```