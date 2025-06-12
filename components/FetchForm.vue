<template>
	<form
		ref="$el"
		:onsubmit="isMounted ? null : 'return false;'"
		:action="action"
		:enctype="enctype"
		:method="method"
		class="c-fetch-form"
		v-bind="attrs"
		@submit="onSubmit"
	>
		<slot v-bind="{ isFetching, currentResponse, currentError }"></slot>
	</form>
</template>

<script setup>
const originalAttrs = useAttrs();
const props = defineProps({
	action: String,
	enctype: {
		type: String,
		default: 'application/x-www-form-urlencoded',
	},
	method: String,

	// The options to pass to the fetch call
	options: {
		type: Object,
		default: () => ({}),
	},

	// Add data before sending it
	dataAppendage: {
		type: Object,
		default: () => ({}),
	},

	// Transform the data before sending it
	dataTransformation: {
		type: Function,
		default: (data) => data,
	},

	useNativeFormDataOnPost: Boolean,

	mockupResponse: null,

	// Disable submitting the form
	disabled: Boolean,
});

const emit = defineEmits(['fetch', 'request', 'response', 'error', 'complete']);

const $el = ref(null);
const isMounted = ref(false);
const method = computed(() => (props.method || 'GET').toUpperCase());
const options = computed(() => {
	return {
		method: method.value,
		...props.options,
		onRequest: (...args) => {
			props.options?.onRequest?.(...args);
			emit('request', ...args);
		},
	};
});

const isFetchingCount = ref(0);
const isFetching = computed(() => isFetchingCount.value > 0);
const currentResponse = ref(null);
const currentError = ref(null);

const attrs = computed(() => {
	const attrs = { ...originalAttrs };
	delete attrs.onSubmit;
	return attrs;
});

onMounted(() => {
	isMounted.value = true;
});

async function onSubmit(e) {
	if (props.disabled) {
		e.preventDefault();
		return;
	}

	// Run submit
	originalAttrs.onSubmit?.(e);
	if (e.defaultPrevented || props.method?.toUpperCase?.() === 'DIALOG') {
		return;
	}

	// Prevent ordinary form handling
	e.preventDefault();

	// Run fetch
	await submit();
}

async function submit(localProps) {
	localProps ??= {};
	localProps = { ...props, ...localProps };

	const localOptions = ref({
		key: String(Math.random()),
		...options.value,
		...localProps.options,
		onRequest: (...args) => {
			localProps.options?.onRequest?.(...args);
			emit('request', ...args);
		},
	});

	if (localProps.action) {
		const actionURL = new URL(localProps.action, 'https://example.com');
		let formData = $el.value?.formData
			? $el.value.formData()
			: new FormData($el.value);
		let origin =
			actionURL.origin === 'https://example.com' ? '' : actionURL.origin;

		// Transform the data
		let payload = formData;
		if (localProps.dataAppendage) {
			for (const [key, value] of Object.entries(
				localProps.dataAppendage
			)) {
				payload.set(key, value);
			}
		}

		if (localProps.dataTransformation) {
			payload = localProps.dataTransformation(payload);
		}

		if (!localProps.useNativeFormDataOnPost) {
			payload = Object.fromEntries(payload);
		}

		// Add query params to GET requests
		if (['GET', 'DELETE'].includes(method.value)) {
			actionURL.search = new URLSearchParams(payload);
		}

		let success = true;
		isFetchingCount.value++;

		// Get mockup response
		if (localProps.mockupResponse) {
			const data =
				typeof localProps.mockupResponse === 'function'
					? localProps.mockupResponse?.(payload)
					: localProps.mockupResponse;
			emit(
				'fetch',
				new Promise((resolve) => {
					emit('response', data);
					emit('complete', true);

					resolve({ meta: { code: 200 }, data, error: null });

					currentResponse.value = localProps.mockupResponse;
					currentError.value = null;
					isFetchingCount.value--;
				})
			);

			return data;
		}

		const fetch = $fetch(origin + actionURL.pathname + actionURL.search, {
			// Add form data to POST requests
			body: ['POST', 'PUT', 'PATCH'].includes(method.value)
				? payload
				: undefined,
			// Add options
			...localOptions.value,
		})
			.then((response) => {
				const { error, data = ref(response) } = response;
				if (error?.value) {
					throw error.value;
				}

				// @response
				emit('response', data.value);

				currentResponse.value = data.value;
				currentError.value = null;
			})
			.catch((error) => {
				// @error
				emit('error', error);
				success = false;

				currentResponse.value = null;
				currentError.value = error;
			})
			.finally(() => {
				// @complete
				isFetchingCount.value--;
				emit('complete', success);
			});

		// @fetch
		emit('fetch', fetch);

		// Await for returning data
		await fetch;
		return currentResponse.value;
	}
}

// Expose fetch function
defineExpose({ isFetching: isFetching.value, submit });
</script>

<script>
export default {
	inheritAttrs: false,
};
</script>
