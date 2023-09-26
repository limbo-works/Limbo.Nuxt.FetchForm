<template>
	<form
		onsubmit="return false;"
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
	enctype: String,
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

	// Disable submitting the form
	disabled: Boolean,
});

const emit = defineEmits(['fetch', 'response', 'error', 'complete']);

const isMounted = ref(false);
const action = computed(() => isMounted.value ? props.action : null);
const method = computed(() => (props.method || 'GET').toUpperCase());
const options = computed(() => ({
	method: method.value,
	...props.options,
}));

const isFetching = ref(false);
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

function onSubmit(e) {
	if (props.disabled) {
		e.preventDefault();
		return;
	}

	// Run submit
	originalAttrs.onSubmit?.(e);
	if (
		e.defaultPrevented ||
      props.method?.toUpperCase?.() === 'DIALOG'
	) {
		return;
	}

	// Run fetch
	if (props.action) {
		const actionURL = new URL(props.action, 'https://example.com');
		let formData = e.target.formData ? e.target.formData() : new FormData(e.target);
		let origin = actionURL.origin === 'https://example.com' ? '' : actionURL.origin;

		// Transform the data
		let payload = formData;
		if (props.dataAppendage) {
			for (const [key, value] of Object.entries(props.dataAppendage)) {
				payload.set(key, value);
			}
		}

		if (props.dataTransformation) {
			payload = props.dataTransformation(payload);
		}

		if (!props.useNativeFormDataOnPost) {
			payload = Object.fromEntries(payload);
		}

		// Add query params to GET requests
		if (method.value === 'GET') {
			actionURL.search = new URLSearchParams(payload);
		}

		let success = true;
		isFetching.value = true;
		const fetch = useFetch(origin + actionURL.pathname + actionURL.search, {
			// Add form data to POST requests
			body: method.value === 'POST' ? payload : undefined,
			// Add options
			...options.value,
		}).then(({ error, data }) => {
			if (error?.value) {
				throw error.value;
			}

			// @response
			emit('response', data.value);

			currentResponse.value = data.value;
			currentError.value = null;
		}).catch((error) => {
			// @error
			emit('error', error);
			success = false;

			currentResponse.value = null;
			currentError.value = error;
		}).finally(() => {
			// @complete
			isFetching.value = false;
			emit('complete', success);
		});

		// @fetch
		emit('fetch', fetch);
	}

	// Prevent ordinary form handling
	e.preventDefault();
}
</script>

<script>
export default {
	inheritAttrs: false,
};
</script>
