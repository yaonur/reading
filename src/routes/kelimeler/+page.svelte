<script lang="ts">
	import Button from '$elements/Button';
	import MultiSelect from 'svelte-multiselect';
	import { onDestroy, onMount } from 'svelte';
	import { browser } from '$app/environment';
    import {words} from "./words"
	let isSpeechSupported =browser && 'speechSynthesis' in window;
	function speak(word:string) {
		if(browser){
			const utterance = new SpeechSynthesisUtterance(word);
			utterance.lang='tr-TR'
			const voices =window.speechSynthesis.getVoices()
			console.log(voices)
			speechSynthesis.speak(utterance);
		}
	}

	let previousWord = '';
	let countdown = 200;
	$: counter = countdown;
    let syllables:[] =[] 

	let timerBar: any = '';
	let createdWord = '';
	let spellingA = '';
	let spellingB = '';
	let interval: any;
	let progressBarInterval: any;
	let isZen = false;
	let gameOn = false;
	let gameFinished = false;
	$: timer = 2000;
	let selectedInterval = 2000;
	
	let selectedVowels: string[] = [];
	let selectedConstants: string[] = [];
	let progressBarTimer = timer;
	let createThreeChar = false;
	let error: any = null;
	if (browser) {
		selectedVowels = localStorage.getItem('selectedVowels')
			? //@ts-ignore
			  JSON.parse(localStorage.getItem('selectedVowels'))
			: [];
		selectedConstants = localStorage.getItem('selectedConstants')
			? //@ts-ignore
			  JSON.parse(localStorage.getItem('selectedConstants'))
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
		let randomIndexWord;
		do {
			randomIndexWord = Math.floor(Math.random() * words.length);
            createdWord = words[randomIndexWord][0]
            syllables = words[randomIndexWord][1]
		} while (createdWord === previousWord);
		counter--;

		previousWord = createdWord;

		if (counter === 0) {
			stopGame();
			// clearInterval(interval);
			// clearInterval(progressBarInterval);
			// progressBarTimer = timer
			// timerBar.style.width = `${(progressBarTimer / timer) * 100}%`;
			gameFinished=true
			// gameOn = false;
		}
	}
	onMount(() => {
		// randomWord();
	});
	function createWord(){
        gameOn=true
        randomWord()
		// speak(createdWord)
    }
	function startGame() {
		gameFinished = false;
		if (
			(selectedVowels.length < 2 || selectedConstants.length === 0) &&
			(selectedVowels.length === 0 || selectedConstants.length < 2)
		) {
			error =
				'Lütfen en az 2 sesli harf ve 1 sessiz harf seçiniz  veya 1 sesli harf ve en az 2 sessiz harf seçiniz';
			return;
		}
		if (!createdWord) randomWord();
		gameOn = true;
		timer = selectedInterval;
		counter = countdown;
		clearInterval(interval);
		clearInterval(progressBarInterval);
		startProgressBar();
		interval = setInterval(
			() => {
				randomWord();
			},
			createThreeChar ? timer * 2 : timer
		);
	}
	function stopGame() {
		createdWord = '';
		gameOn = false;
		counter = countdown;
		progressBarTimer = timer;
		timerBar = document.getElementById('timer-bar');
		timerBar.style.width = `${(progressBarTimer / timer) * 100}%`;
		clearInterval(interval);
		clearInterval(progressBarInterval);
	}
	function startProgressBar() {
		progressBarTimer = timer;
		progressBarInterval = setInterval(() => {
			// @ts-ignore
			timerBar = document.getElementById('timer-bar');
			timerBar.style.width = `${(progressBarTimer / timer) * 100}%`;
			progressBarTimer -= 20;
			if (progressBarTimer === 0) progressBarTimer = timer;
		}, 20);
	}
	onDestroy(() => {
		clearInterval(interval);
		clearInterval(progressBarInterval);
	});
</script>

<div
	class="flex h-screen-minus-navbar flex-col items-center {isZen
		? 'justify-center gap-8'
		: 'justify-around'} bg-slate-300 sm:h-full sm:min-h-screen"
>
	{#if error}
		<p class="text-red-500">{error}</p>
	{/if}
	{#if gameOn}
		<div class="flex flex-col items-center">
			<p class="text-9xl leading-tight underline transition-all duration-1000 ease-linear">
				{createdWord}
			</p>
			<div class="flex gap-16 text-6xl font-light leading-none underline">
				{#each syllables as syllable }
                    <p>{syllable}</p>
                {/each}
			</div>
		</div>
	{:else if gameFinished}
		<div>
			<p>Tebrikler Bitirdin!</p>
		</div>
	{/if}
	<div class="py-4 {isZen ? 'opacity-0' : 'opacity-100'}">
		{#if gameOn}
			<p class="text-center">Kalan Kelime:{counter}</p>
		{/if}
		<div class="h-1 bg-red-500">
			<div id="timer-bar" class="h-full bg-green-500" />
		</div>
		<div>
			<input
				class="w-full"
				type="range"
				min="500"
				max="15000"
				step="100"
				bind:value={selectedInterval}
			/>
			<p>Aralık:{selectedInterval / 1000} saniye</p>
		</div>
		<div class="flex items-center gap-4">
			<Button on:click={startGame} class="h-6 w-16 rounded-xl   bg-blue-500">Başla</Button>
			<Button on:click={stopGame} class="h-6 w-16 rounded-xl">Bitir</Button>
			<div class="flex flex-col">
				<input type="range" min="5" max="500" step="5" bind:value={countdown} />
				<label class="leading-none">Tekrar Sayısı: {countdown}</label>
				<label>
					<input type="checkbox" bind:checked={isZen} disabled={!gameOn} />
					Odak Modu
				</label>
			</div>
		</div>
		<div>
            <Button class="bg-green-600 w-full" on:click={createWord}>Elle Türet</Button>
        </div>
        <div class="flex gap-4 mt-4">
            <Button class="bg-blue-600 w-full" on:click={()=>countdown=20}>20</Button>
            <Button class="bg-blue-600 w-full" on:click={()=>countdown=40}>40</Button>
            <Button class="bg-blue-600 w-full" on:click={()=>countdown=60}>60</Button>
        </div>
	</div>
	{#if isZen}
		<label class="absolute top-10">
			<input type="checkbox" bind:checked={isZen} />
			Odak Modu
		</label>
	{/if}
</div>
