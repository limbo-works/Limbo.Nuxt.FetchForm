# FetchForm

Vue component for a form that uses fetch under the hood, but otherwise looks like a standard form element.

## Installation

``` bash
yarn add @limbo-works/fetch-form
```

## Using the component

Make the component globally usable by extending the layer in `nuxt.config.js`.

``` js
export default defineNuxtConfig({
    extends: [
        '@limbo-works/fetch-form',
        ...
    ],
    ...
});
```

Then you can use the `FetchForm` component anywhere within that solution:

``` html
<!-- As written in Vue -->
<FetchForm
    action="/some/endpoint"

>
...
</FetchForm>

<!-- As it may appear in the dom -->
<form action="/some/endpoint" method="GET">
...
</form>
```

### Props overview

| Prop | Description | Default value | Data type |
| ---- | ----------- | ------------- | --------- |
| options | Options to pass the fetch request. | {} | Object |
| dataAppendage | An object with extra key-value pairs for the request, on top of whatever named form fields inside the form. | {} | Object |
| <span class="colour" style="color:rgb(225, 228, 232)"></span>dataTransformation | A transformer-function to change the data before performing the request. | (data) => data | Function |
| useNativeFormDataOnPost | If false the form (on POST) will send the data as form data. If true a JSON object will be send instead. | false | Boolean |

Further you'll of course set an `action`, can set the `method` (will default to "GET" if not set) and can `disable` using the standard form attributes.

### Events overview

| Event | Description |
| ----- | ----------- |
| @response | Emits when the form request returns with a non-error response. Includes the response data. |
| @error | Emits when the form request returns with an error. Includes the error. |
| @complete | Emits when the form request returns. Includes a boolean value of whether the request were succesful or not. |
| @fetch | Emits when a new request is made. Includes the promise of the request itself. |

### Exposed slot props

| Prop | Description |
| ---- | ----------- |
| isFetching | Boolean value for whether its fetching or not. |
| currentResponse | If the latest fetch gave a response, this will have its value. |
| currentError | If the latest fetch gave an error, this will have its value. |