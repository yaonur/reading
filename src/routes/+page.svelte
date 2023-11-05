<script lang="ts">
	import Button from '$elements/Button';
	import MultiSelect from 'svelte-multiselect';
	import { onDestroy, onMount } from 'svelte';
	import { browser } from '$app/environment';

	let previousWord = '';
	let countdown = 200;

	let createdWord = '';
	let spellingA = '';
	let spellingB = '';
	let interval: any;
	let progressBarInterval: any;
	$: timer = 2000;
	let selectedInterval = 2000
	const vowels = ['a', 'e', 'ı', 'i', 'o', 'ö', 'u', 'ü'];
	const constants = [
		'b',
		'c',
		'ç',
		'd',
		'f',
		'g',
		'ğ',
		'h',
		'j',
		'k',
		'l',
		'm',
		'n',
		'p',
		'r',
		's',
		'ş',
		't',
		'v',
		'y',
		'z'
	];
	let selectedVowels: string[] = [];
	let selectedConstants: string[] = [];
	let startWithVowel = true;
	let createThreeChar = false;
	let error = null;
	if (browser) {
		selectedVowels = localStorage.getItem('selectedVowels')
			? JSON.parse(localStorage.getItem('selectedVowels'))
			: [];
		selectedConstants = localStorage.getItem('selectedConstants')
			? JSON.parse(localStorage.getItem('selectedConstants'))
			: [];
	}

	$: {
		if (browser) {
			localStorage.setItem('selectedVowels', JSON.stringify(selectedVowels));
			localStorage.setItem('selectedConstants', JSON.stringify(selectedConstants));
		}
	}

	function sleep() {
		return new Promise((resolve) => setTimeout(resolve, timer));
	}

	async function randomWord() {
		error = null;
		if (
			(selectedVowels.length < 2 || selectedConstants.length === 0) &&
			(selectedVowels.length === 0 || selectedConstants.length < 2)
		) {
			error =
				'Lütfen en az 2 sesli harf ve 1 sessiz harf seçiniz  veya 1 sesli harf ve en az 2 sessiz harf seçiniz';
			return;
		}
		let randomIndexVowel;
		let randomIndexConstant;
		let vowel;
		let constant;
		do {
			randomIndexVowel = Math.floor(Math.random() * selectedVowels.length);
			randomIndexConstant = Math.floor(Math.random() * selectedConstants.length);

			vowel = selectedVowels[randomIndexVowel];
			constant = selectedConstants[randomIndexConstant];
			createdWord = startWithVowel ? vowel + constant : constant + vowel;
			startWithVowel ? (spellingA = vowel) : (spellingA = constant);
			startWithVowel ? (spellingB = constant) : (spellingB = vowel);
			// if (countdown === 0) {
			// 	break;
			// }
		} while (createdWord === previousWord);
		countdown--;

		previousWord = createdWord;

		if (createThreeChar) {
			await sleep();
			createdWord = createdWord + spellingA;
			spellingA = previousWord;
			startWithVowel ? (spellingB = constant) : (spellingB = vowel);
		}
		if (countdown === 0) {
			clearInterval(interval);
		}
	}
	onMount(() => {
		// randomWord();
	});
	function startGame() {
		if (!createdWord) randomWord();
		clearInterval(interval);
		startProgressBar();
		interval = setInterval(
			() => {
				randomWord();
			},
			createThreeChar ? timer * 2 : timer
		);
			
	}
	function stopGame() {
		clearInterval(interval);
		clearInterval(progressBarInterval);
	}
	function startProgressBar(){
		const timerBar = document.getElementById('timer-bar');
		let progressBarTimer=timer
		progressBarTimer=timer
		progressBarInterval = setInterval(() => {
			timerBar.style.width = `${(progressBarTimer/timer) * 100}%`;
			progressBarTimer-=20
			if(progressBarTimer===0) progressBarTimer=timer
		}, 20);
	}
	onDestroy(() => {
		clearInterval(interval);
		clearInterval(progressBarInterval);
	});
</script>

<div class="flex h-[100vh] flex-col items-center justify-between bg-slate-300 pt-8">
	<div class="flex flex-col gap-4">
		<div>
			<p>Sesli Harfler</p>
			<p />
			<MultiSelect
				placeholder="Sesli Harf Ekle"
				bind:selected={selectedVowels}
				options={vowels}
				outerDivClass="border border-red-300"
			/>
		</div>
		<div>
			<p>Sessiz Harfler</p>
			<MultiSelect
				placeholder="Sessiz Harf Ekle"
				bind:selected={selectedConstants}
				options={constants}
				outerDivClass="border border-red-300"
			/>
		</div>
		<div>
			<label>
				<input type="checkbox" bind:checked={startWithVowel} />
				Sesli Harfle Başlasın
			</label>
			<label>
				<input
					type="checkbox"
					bind:checked={createThreeChar}
					on:change={() => {
						clearInterval(interval);
						startGame();
					}}
				/>
				3 Harfli Kelime Türet
			</label>
		</div>
	</div>
	{#if error}
		<p class="text-red-500">{error}</p>
	{/if}
	<div class="relative flex flex-col items-center">
		<p class="text-[266px] leading-none underline">{createdWord}</p>
		<div class="flex gap-24 text-6xl font-light leading-none underline">
			<p>{spellingA}</p>
			<p>{spellingB}</p>
		</div>
	</div>
	<div class="p-16">
		<div class="h-4 bg-red-500">
			<div id="timer-bar"  class="h-full bg-green-500" />
		</div>
		<input type="range" min="500" max="15000" step="100" bind:value={selectedInterval} />
		<p>Aralık:{selectedInterval/1000} saniye </p>
		<div class="flex gap-4">
			<Button on:click={startGame} class="w-12">Start</Button>
			<Button on:click={stopGame} class="w-12">Stop</Button>
		</div>
		<p class="text-center text-4xl">{countdown}</p>
	</div>
</div>
