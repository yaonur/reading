<script lang="ts">
	import Button from '$elements/Button';
	import MultiSelect from 'svelte-multiselect';
	import { onDestroy, onMount } from 'svelte';
	import { browser } from '$app/environment';

	let previousWord = '';
	let countdown = 200;
	let counter = countdown;

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
    let selectedFirstChars: string[] = [];
    let selectedSecondChars: string[] = [];
    let selectedThirdChars: string[] = [];
    let isVowelFirst = true;
    let isVowelSecond = true;
    let isVowelThird = true;
	let progressBarTimer = timer;
	let startWithVowel = true;
	let createThreeChar = false;
	let error: any = null;
	if (browser) {
		selectedFirstChars = localStorage.getItem('selectedFirstChars')
			? //@ts-ignore
			  JSON.parse(localStorage.getItem('selectedFirstChars'))
			: [];
		selectedSecondChars = localStorage.getItem('selectedSecondChars')
			? //@ts-ignore
			  JSON.parse(localStorage.getItem('selectedSecondChars'))
			: [];
        selectedThirdChars = localStorage.getItem('selectedThirdChars')
			? //@ts-ignore
			  JSON.parse(localStorage.getItem('selectedThirdChars'))
			: [];
	}

	$: {
		if (browser) {
			localStorage.setItem('selectedFirstChars', JSON.stringify(selectedFirstChars));
			localStorage.setItem('selectedSecondChars', JSON.stringify(selectedSecondChars));
			localStorage.setItem('selectedThirdChars', JSON.stringify(selectedThirdChars));
		}
	}

	function sleep() {
		return new Promise((resolve) => setTimeout(resolve, timer));
	}

	async function randomWord() {
		error = null;
		let randomIndexFirstChar;
		let randomIndexSecondChar;
		let randomIndexThirdChar;
		let firstChar;
		let secondChar;
		let thirdChar;
		do {
			randomIndexFirstChar = Math.floor(Math.random() * selectedFirstChars.length);
			randomIndexSecondChar = Math.floor(Math.random() * selectedSecondChars.length);
			randomIndexThirdChar = Math.floor(Math.random() * selectedThirdChars.length);

			firstChar = selectedFirstChars[randomIndexFirstChar];
			secondChar = selectedSecondChars[randomIndexSecondChar];
			thirdChar = selectedThirdChars[randomIndexThirdChar];
			createdWord = firstChar + secondChar + thirdChar;
            spellingA = firstChar+secondChar;
            spellingB = thirdChar;
			// if (countdown === 0) {
			// 	break;
			// }
		} while (createdWord === previousWord);
		counter--;

		previousWord = createdWord;

		if (counter === 0) {
			stopGame();
			// clearInterval(interval);
			// clearInterval(progressBarInterval);
			// progressBarTimer = timer
			// timerBar.style.width = `${(progressBarTimer / timer) * 100}%`;
			// gameFinished=true
			// gameOn = false;
		}
	}
	onMount(() => {
		// randomWord();
	});
    function createWord(){
        gameOn=true
        randomWord()
    }
	function startGame() {
		gameFinished = false;
		
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
	class="flex h-screen-minus-navbar overflow-auto flex-col items-center {isZen
		? 'justify-center space-y-8'
		: 'justify-around'} bg-slate-300 sm:h-full sm:min-h-screen"
>
	{#if !isZen && !gameOn}
		<div class="flex flex-col gap-4">
			<div>
				<p>Birinci Harf</p>
                <label>
					<input type="checkbox" bind:checked={isVowelFirst} />
					Sesli Harfler 
				</label>
				<MultiSelect
					placeholder="Sesli Harf Ekle"
					bind:selected={selectedFirstChars}
					options={isVowelFirst?vowels:constants}
					outerDivClass="border border-red-300"
				/>
			</div>
			<div>
				<p>Ikinci harf</p>
                <label>
					<input type="checkbox" bind:checked={isVowelSecond} />
					Sesli Harfler 
				</label>
				<MultiSelect
					placeholder="Sessiz Harf Ekle"
					bind:selected={selectedSecondChars}
					options={isVowelSecond?vowels:constants}
					outerDivClass="border border-red-300"
				/>
			</div>
            <div>
				<p>Ucuncu  Harf</p>
                <label>
					<input type="checkbox" bind:checked={isVowelThird} />
					Sesli Harfler 
				</label>
				<MultiSelect
					placeholder="Sessiz Harf Ekle"
					bind:selected={selectedThirdChars}
					options={isVowelThird?vowels:constants}
					outerDivClass="border border-red-300"
				/>
			</div>
			<div>
				<label>
					<input type="checkbox" bind:checked={startWithVowel} />
					Sesli Harfle Başlasın
				</label>
				<label>
					<input type="checkbox" bind:checked={createThreeChar} />
					3 Harfli Kelime Türet
				</label>
			</div>
		</div>
	{/if}
	{#if error}
		<p class="text-red-500">{error}</p>
	{/if}
	{#if gameOn}
		<div class="flex flex-col items-center">
			<p class="text-9xl leading-tight underline transition-all duration-1000 ease-linear">
				{createdWord}
			</p>
			<div class="flex gap-16 text-6xl font-light leading-none underline">
				<p class="text-red-600">{spellingA}</p>
				<p class="text-blue-600">{spellingB}</p>
			</div>
		</div>
	{:else if gameFinished}
		<div>
			<p>Tebrikler Bitirdin!</p>
		</div>
	{/if}
	<div class="py-4 {isZen ? 'opacity-0' : 'opacity-100'}">
		{#if gameOn}
			<p class="text-center text-6xl">Kalan Kelime:{counter}</p>
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
			<Button on:click={stopGame} class="h-6 w-16 rounded-xl  ">Bitir</Button>
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
	</div>
	{#if isZen}
		<label class="absolute top-10">
			<input type="checkbox" bind:checked={isZen} />
			Odak Modu
		</label>
	{/if}
</div>
