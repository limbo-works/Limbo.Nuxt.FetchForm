<template>
	<div>
		<FetchForm
			:action="`https://pokeapi.co/api/v2/pokemon/${pokemon.toLowerCase()}`"
			#default="{ isFetching, currentResponse, currentError }"
			:mockup-response="(payload) => ({ name: 'ditto', payload })"
			@response="onResponse"
			@error="onError"
		>
			<input v-model="pokemon" type="text" name="pokemon" placeholder="Enter a pokemon name" />
			<button type="submit">Fetch</button>
			<div v-if="isFetching">Fetching...</div>
			<div v-if="currentError">Error!</div>
			<div v-if="currentResponse">Has data!</div>
		</FetchForm>
		<pre>{{ data }}</pre>
	</div>
</template>

<script setup>
const pokemon = ref('ditto');
const data = ref(null);

const onResponse = (response) => {
	data.value = response;
};
const onError = (error) => {
	data.value = 'No data';
};
</script>
