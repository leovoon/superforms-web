<script lang="ts">
  import Head from '$lib/Head.svelte'
  import Form from './Form.svelte'
  import Next from '$lib/Next.svelte'
  import Flowchart from './Flowchart.svelte'
	import SuperDebug from 'sveltekit-superforms/client/SuperDebug.svelte'
  import { concepts } from '$lib/navigation/sections'

	export let data;
</script>

# Events

<Head title="Events" />

> Events are only available when JavaScript is enabled in the browser and the custom [use:enhance](/concepts/enhance) is added to the form.

A number of events, triggered on form submission, give you full control over the submit process. They are triggered every time the form is submitted.

## Event flowchart

<Flowchart />

> In a [single-page application](/concepts/spa), or if [client-side validation](/concepts/client-validation) fails, the validation result, usually returned from the server between `onSubmit` and `onResult`, is created on the client.

## Usage

```ts
const { form, enhance } = superForm(data.form, {
  onSubmit: ({ action, formData, formElement, controller, submitter, cancel }) => void
  onResult: ({ result, formEl, cancel }) => void
  onUpdate: ({ form, cancel }) => void
  onUpdated: ({ form }) => void
  onError: (({ result, message }) => void) | 'apply'
})
```

### onSubmit

```ts
onSubmit: ({
  action,
  formData,
  formElement,
  controller,
  submitter,
  cancel
}) => void;
```

The `onSubmit` event is the first one triggered, hooking you into SvelteKit's own `use:enhance` function, and also giving you a chance to cancel the submission altogether. See the SvelteKit docs for the [SubmitFunction](https://kit.svelte.dev/docs/types#public-types-submitfunction) signature.

### onResult

```ts
onResult: ({ result, formEl, cancel }) => void
```

If the submission isn't cancelled, the form is posted to the server. It then responds with a SvelteKit [ActionResult](https://kit.svelte.dev/docs/types#public-types-actionresult), triggering the `onResult` event.

`result` contains the ActionResult. You can modify it; changes will be applied further down the event chain. `formEl` is the `HTMLFormElement` of the form. `cancel()` is a function which will cancel the rest of the event chain and any form updates.

> Usually, you don't have to care about the `ActionResult` itself. To check if the form validation succeeded, use [onUpdated](/concepts/events#onupdated) instead, or [onUpdate](/concepts/events#onupdate) if you want to modify or cancel the result before it's displayed.

#### Common usage

`onResult` is useful when you have set `applyAction = false` and still want to redirect, since the form doesn't do that automatically in that case. Then you can do:

```ts
const { form, enhance } = superForm(data.form, {
  applyAction: false,
  onResult({ result }) {
    if (result.type === 'redirect') {
      goto(result.location);
    }
  }
});
```

Also, if `applyAction` is `false`, which means that `$page.status` won't update, you'll find the status code for the request in `result.status`.

#### Strongly typed ActionResult

Usually, you check the ActionResult status in `onResult`, not the form validation result, which is more conveniently handled in `onUpdate` (see below). But if you return additional data in the form action, there is a helper type called `FormResult`, that you can use to make the ActionResult data strongly typed:

```svelte
<script lang="ts">
  import { superForm, type FormResult } from 'sveltekit-superforms/client';
  import type { PageData, ActionData } from './$types';

  export let data: PageData;

  const { form, enhance } = superForm(data.form, {
    onResult: (event) => {
      const result = event.result as FormResult<ActionData>;
      if (result.type == 'failure') {
        // Strongly typed now:
        console.log(result.data?.form.data.name);
      }
    }
  });
</script>
```

### onUpdate

```ts
onUpdate: ({ form, formEl, cancel }) => void
```

The `onUpdate` event is triggered just before the form update is being applied, giving you the option to modify the validation result in `form`, or use `cancel()` to negate the update altogether. You also have access to the form's `HTMLFormElement` with `formEl`. 

> If your app is a single-page application, you should use `onUpdate` to process the form data. See the [SPA](/concepts/spa) page for more details.

When you don't want to modify or cancel the validation result, the last event is the most convenient to use:

### onUpdated

```ts
onUpdated: ({ form }) => void
```

When you just want to ensure that the form is validated and do something extra afterwards, like showing a toast notification, `onUpdated` is the easiest way.

The `form` parameter contains the validation result, and is read-only here, since the stores have updated at this point.

**Example**

```ts
const { form, enhance } = superForm(data.form, {
  onUpdated({ form }) {
    if (form.valid) {
      // Successful post! Do some more client-side stuff,
      // like showing a toast notification.
      toast(form.message, { icon: '✅' });
    }
  }
});
```

### onError

```ts
onError: (({ result }) => void) | 'apply'
```

When the SvelteKit [error](https://kit.svelte.dev/docs/errors#expected-errors) function is thrown on the server, you can use the `onError` event to catch it.

`result` is the error ActionResult, with the `error` property casted to [App.Error](https://kit.svelte.dev/docs/types#app-error). In this simple example, the message store is set to the error:

```ts
const { form, enhance, message } = superForm(data.form, {
  onError({ result }) {
    $message = result.error.message;
  }
});
```

You can also set `onError` to the string value `'apply'`, in which case the SvelteKit `applyAction` error behaviour will be used, which is to render the nearest [+error](https://kit.svelte.dev/docs/routing#error) page, also wiping out the form. To avoid data loss even for non-javascript users, returning a [status message](/concepts/messages) instead of throwing an error is recommended.

> If you're unsure what event to use, start with `onUpdated`; unless your app is a [SPA](/concepts/spa), then `onUpdate` should be used to validate the form data.

<Next section={concepts} />
