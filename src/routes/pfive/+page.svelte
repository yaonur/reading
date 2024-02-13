<script lang="ts">
	import { scale } from 'svelte/transition';
	import type { PageData } from './$types';

	export let data: PageData;
	let target = 10;
	let num = 6;
	let isFinished = false;
	const randomNum = () => {
		let oldNum = num;
		num = Math.floor(Math.random() * 4) + 6;
		// console.log(Math.random());
		if (num === oldNum) randomNum();
	};
	function handleAnswer(e, answer) {
		console.log(e.target);
		if (answer === num - 5) {
			target--;
			e.target.classList.add('celebrate');
			setTimeout(() => e.target.classList.remove('celebrate'), 1000);
			if (target === 0) {
				isFinished = true;
				return;
			}
			randomNum();
		}
	}
	function handleRestart() {
		target = 10;
		num = 6;
		isFinished = false;
	}
</script>

<div class="flex flex-col justify-center px-44 py-10 gap-4 text-2xl">
	{#if isFinished}
		<p>Tebrikler Bitirdin!!!</p>
		<button class="h-8 w-20 bg-green-400" on:click={handleRestart}>Yeniden Basla</button>
	{:else}
		<p class="text-6xl">{num}=5+x</p>
		<p class="text-6xl">x=?</p>
		<div class="flex gap-4">
			<button class={'bg-blue-300 w-20'} on:click={(e) => handleAnswer(e, 1)}>1</button>
			<button class={'bg-blue-300 w-20'} on:click={(e) => handleAnswer(e, 2)}>2</button>
			<button class={'bg-blue-300 w-20'} on:click={(e) => handleAnswer(e, 3)}>3</button>
			<button class={'bg-blue-300 w-20'} on:click={(e) => handleAnswer(e, 4)}>4</button>
		</div>
		<button class="h-8 w-40 bg-green-400" on:click={randomNum}>Siradaki</button>
		<div>
			<input type="range" min="10" max="400" bind:value={target} step="10" />
			<p>Kalan= {target}</p>
		</div>
		<!-- Dummy element to prevent Svelte from removing the 'celebrate' class -->
		<div class="celebrate" style="display: none;"></div>
	{/if}
</div>

<style>
	button:active {
		transform: scale(0.8);
		transition: transform 0.15s;
	}
	.celebrate {
		animation: celebrate 1.5s;
	}

	@keyframes celebrate {
		0%,
		100% {
			transform: scale(1);
			background-color: rgb(147 197 253); /* Initial color */
		}
		50% {
			transform: scale(1.4);
			background-color: #4ff13d; /* Color at the midpoint of the animation */
		}
	}
</style>
