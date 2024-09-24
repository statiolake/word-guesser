<script lang="ts">
	import Confetti from '$lib/components/Confetti.svelte';
	import Input from '$lib/components/Input.svelte';
	import { CheckCircle2, XCircle } from 'lucide-svelte';
	import { onMount } from 'svelte';
	import { fade, scale } from 'svelte/transition';

	const words: string[] = [
		'javascript',
		'react',
		'component',
		'function',
		'state',
		'effect',
		'props',
		'hook',
		'context',
		'reducer'
	];

	let targetWord: string = '';
	let guess: string = '';
	let matchStatus: 'match' | 'no-match' | 'correct' | 'error' | null = null;
	let gameOver: boolean = false;
	let showConfetti: boolean = false;
	let attempts: number = 0;
	let inputElement: Input;

	function startNewGame(): void {
		targetWord = words[Math.floor(Math.random() * words.length)];
		guess = '';
		matchStatus = null;
		gameOver = false;
		showConfetti = false;
		attempts = 0;
		if (inputElement) inputElement.focus();
	}

	function handleGuess(): void {
		attempts++;
		try {
			const regex = new RegExp(`^${guess}$`);
			if (regex.test(targetWord)) {
				if (guess === targetWord) {
					gameOver = true;
					matchStatus = 'correct';
					showConfetti = true;
				} else {
					matchStatus = 'match';
				}
			} else {
				matchStatus = 'no-match';
			}
		} catch (error) {
			matchStatus = 'error';
		}
	}

	function handleKeyPress(e: CustomEvent<KeyboardEvent>): void {
		if (e.detail.key === 'Enter' && !gameOver) {
			handleGuess();
		}
	}

	function getBackgroundColor(status: typeof matchStatus): string {
		switch (status) {
			case 'match':
			case 'correct':
				return 'bg-green-100';
			case 'no-match':
			case 'error':
				return 'bg-red-100';
			default:
				return 'bg-gray-300';
		}
	}

	onMount(() => {
		startNewGame();
	});
</script>

<div
	class="flex items-center justify-center min-h-screen transition-colors duration-300 {getBackgroundColor(
		matchStatus
	)} overflow-hidden"
>
	{#if showConfetti}
		<Confetti />
	{/if}

	<div class="relative w-full max-w-md px-4">
		<span class="absolute left-7 top-1/2 transform -translate-y-1/2 text-gray-600 text-3xl">/</span>
		<Input
			type="text"
			bind:value={guess}
			on:keypress={handleKeyPress}
			placeholder="Enter your regex query..."
			disabled={gameOver}
			bind:this={inputElement}
		/>
		<span class="absolute right-7 top-1/2 transform -translate-y-1/2 text-gray-600 text-3xl">/</span
		>
		{#if matchStatus}
			<span class="absolute right-16 top-1/2 transform -translate-y-1/2">
				{#if matchStatus === 'match' || matchStatus === 'correct'}
					<CheckCircle2 class="text-green-600" size={32} />
				{:else}
					<XCircle class="text-red-600" size={32} />
				{/if}
			</span>
		{/if}
	</div>

	{#if !gameOver}
		<div
			class="fixed bottom-4 right-4 bg-gray-800 text-white px-3 py-1 rounded-full text-lg"
			in:scale={{ duration: 300, start: 0.5 }}
			out:fade
		>
			{attempts}
		</div>
	{/if}

	{#if gameOver}
		<div
			class="absolute inset-0 flex items-center justify-center"
			in:scale={{ duration: 500, start: 0.5 }}
			out:fade
		>
			<div class="bg-white bg-opacity-90 rounded-2xl p-8 shadow-2xl text-center">
				<h2 class="text-4xl font-bold mb-4">Congratulations!</h2>
				<p class="text-2xl mb-4">You guessed the word</p>
				<p
					class="text-4xl font-bold text-indigo-600 mb-6 px-4 py-2 bg-indigo-100 rounded-lg inline-block"
				>
					{targetWord}
				</p>
				<p class="text-2xl mb-2">in</p>
				<p class="text-6xl font-bold text-indigo-600 mb-2">{attempts}</p>
				<p class="text-2xl mb-6">attempts</p>
				<button
					on:click={startNewGame}
					class="bg-indigo-600 text-white px-6 py-3 rounded-full text-xl hover:bg-indigo-700 transition-colors"
				>
					Play Again
				</button>
			</div>
		</div>
	{/if}
</div>
