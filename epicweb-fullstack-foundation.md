## 1. Styling
- Assets import: Remix caches static asset for 1y (depending on our server). If we update the assets, we also need to update the asset name (cache key), so browser can download the asset again. Another better solution is to import the asset and Remix will take care of getting it into the `public` directory and give us the URL to it. As part of this process, Remix will fingerprint the file. 
```
import faviconAssetUrl from './assets/favicon.svg'
```
- Remix has built-in supports for Tailwind + PostCSS
## Data loading
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
## Data mutation
- Broswer form only supports FETCH and POST. Other actions will be fallback to GET.
- Form with GET action will seriallize inputs into a "query string"
- Form with POST action will submit the form body as a "payload". The browser will encode it as `application/x-www-form-urlencoded`