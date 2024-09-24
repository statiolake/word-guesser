<script lang="ts">
	import { createEventDispatcher } from 'svelte';

	export let type = 'text';
	export let value = '';
	export let placeholder = '';
	export let disabled = false;
	export let className = '';

	const dispatch = createEventDispatcher<{
		input: { value: string };
		keypress: KeyboardEvent;
	}>();

	let inputElement: HTMLInputElement;

	export function focus() {
		inputElement.focus();
	}

	function onInput(_event: Event) {
		value = inputElement.value;
		dispatch('input', { value });
	}

	function onKeyPress(event: KeyboardEvent) {
		dispatch('keypress', event);
	}
</script>

<input
	bind:this={inputElement}
	{type}
	{placeholder}
	{disabled}
	{value}
	on:input={onInput}
	on:keypress={onKeyPress}
	class={`pl-10 pr-12 py-6 w-full bg-white bg-opacity-20 border-2 border-gray-600 rounded-lg text-gray-800 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-gray-600 focus:border-transparent text-xl ${className}`}
	{...$$restProps}
/>
