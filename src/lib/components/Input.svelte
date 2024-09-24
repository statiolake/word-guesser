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
	let isFocused = false;

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

	function onFocus() {
		isFocused = true;
	}

	function onBlur() {
		isFocused = false;
	}
</script>

<div class="relative">
	<input
		bind:this={inputElement}
		{type}
		{placeholder}
		{disabled}
		{value}
		on:input={onInput}
		on:keypress={onKeyPress}
		on:focus={onFocus}
		on:blur={onBlur}
		class={`
			pl-10 pr-12 py-6 w-full
			bg-white bg-opacity-20
			border-2 border-gray-600
			rounded-lg text-gray-800
			placeholder-gray-500
			focus:outline-none
			transition-all duration-200 ease-in-out
			${className}
		`}
		{...$$restProps}
	/>
	<div
		class="
			absolute inset-0
			rounded-lg
			pointer-events-none
			transition-all duration-200 ease-in-out
			{isFocused ? 'ring-2 ring-gray-600' : ''}
		"
	></div>
</div>
