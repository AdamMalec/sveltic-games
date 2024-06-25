<script lang="ts">
	import Dialog from '$lib/components/Dialog.svelte';
	import { onMount } from 'svelte';
	import { tweened } from 'svelte/motion';
	import { blur } from 'svelte/transition';

	type Game = 'waiting for input' | 'in progress' | 'game over' | 'paused';
	type Word = string;

	let game: Game = 'waiting for input';
	let seconds = 60;
	let typedLetter = '';

	let words: Word[] =
		`If you really want to hear about it, the first thing you’ll probably want to know is where I was born, an what my lousy childhood was like, and how my parents were occupied and all before they had me, and all that David Copperfield kind of crap, but I don’t feel like going into it, if you want to know the truth. In the first place, that stuff bores me, and in the second place, my parents would have about two hemorrhages apiece if I told anything pretty personal about them. They’re quite touchy about anything like that, especially my father. They’re nice and all—I’m not saying that—but they’re also touchy as hell. Besides, I’m not going to tell you my whole goddam autobiography or anything. I’ll just tell you about this madman stuff that happened to me around last Christmas just before I got pretty run-down and had to come out here and take it easy. I mean that’s all I told D.B. about, and he’s my brother and all. He’s in Hollywood. That isn’t too far from this crumby place, and he comes over and visits me practically every week end. He’s going to drive me home when I go home next month maybe. He just got a Jaguar. One of those little English jobs that can do around two hundred miles an hour. It cost him damn near four thousand bucks. He’s got a lot of dough, now. He didn’t use to. He used to be just a regular writer, when he was home. He wrote this terrific book of short stories, The Secret Goldfish, in case you never heard of him. The best one in it was “The Secret Goldfish.” It was about this little kid that wouldn’t let anybody look at his goldfish because he’d bought it with his own money. It killed me. Now he’s out in Hollywood, D.B., being a prostitute. If there’s one thing I hate, it’s the movies. Don’t even mention them to me.`.split(
			' '
		);
	let wordIndex = 0;
	let letterIndex = 0;
	let correctLetters = 0;
	let toggleReset = false;

	let wordsPerMinute = tweened(0, { delay: 200, duration: 500 });
	let precision = tweened(0, { delay: 200, duration: 500 });

	let wordsEl: HTMLDivElement;
	let letterEl: HTMLSpanElement | undefined;
	let inputEl: HTMLInputElement;
	let caretEl: HTMLDivElement;

	let dialog: HTMLDialogElement;
	let messageLayout: HTMLElement;

	const isWordCompleted = () => letterIndex > words[wordIndex].length - 1;
	const isOneLetterWord = () => words[wordIndex].length === 1;

	function resetGame() {
		inputEl.disabled = false;
		toggleReset = !toggleReset;
		letterEl = undefined;

		setGameState('waiting for input');
		messageLayout.classList.remove('info-layout--active')
		seconds = 60;
		typedLetter = '';
		wordIndex = 0;
		letterIndex = 0;
		correctLetters = 0;

		$wordsPerMinute = 0;
		$precision = 0;

		focusInput();
	}

	function updateGameState() {
		setLetter();
		checkLetter();
		nextLetter();
		resetLetter();
		updateLine();
		moveCaret();
	}

	function setLetter() {
		if (!isWordCompleted()) {
			letterEl = wordsEl.children[wordIndex].children[letterIndex] as HTMLSpanElement;
		}
	}

	function checkLetter() {
		const currentLetter = words[wordIndex][letterIndex];

		if (typedLetter.codePointAt(0)! > 127) {
			messageLayout.classList.add('info-layout--active');
		} else messageLayout.classList.remove('info-layout--active');

		if (letterEl !== undefined) {
			if (typedLetter === currentLetter) {
				letterEl.dataset.letter = 'correct';
				increaseScore();
			}

			if (typedLetter !== currentLetter) {
				letterEl.dataset.letter = 'incorrect';
			}
		}
	}

	function increaseScore() {
		correctLetters += 1;
	}

	function nextLetter() {
		letterIndex += 1;
	}

	function resetLetter() {
		typedLetter = '';
	}

	function updateLine() {
		const wordEl = wordsEl.children[wordIndex];
		const wordsY = wordsEl.getBoundingClientRect().y;
		const wordY = wordEl.getBoundingClientRect().y;

		if (wordY > wordsY) {
			wordEl.scrollIntoView({ block: 'center' });
		}
	}

	function nextWord() {
		if (isOneLetterWord() || isWordCompleted()) {
			wordIndex += 1;
			letterIndex = 0;
			setLetter();
			moveCaret('whitespace');
		}
	}

	function moveCaret(whitespace?: string) {
		const offset = 2;
		const { offsetTop, offsetLeft, offsetWidth } = letterEl!;

		caretEl.style.top = `${offsetTop}px`;

		if (whitespace) {
			caretEl.style.left = `${offsetLeft - offset}px`;
		} else {
			caretEl.style.left = `${offsetLeft + offsetWidth - offset}px`;
		}
	}

	function startGame() {
		setGameState('in progress');
		setGameTimer();
	}

	function setGameState(state: Game) {
		game = state;
	}

	function setGameTimer() {
		function gameTimer() {
			if (game !== 'paused' && seconds > 0) {
				seconds -= 1;
			}

			if (game === 'waiting for input' || seconds === 0) {
				clearInterval(interval);
			}

			if (seconds === 0) {
				inputEl.disabled = true;
				setGameState('game over');
				getResults();
				dialog.showModal();
			}
		}

		const interval = setInterval(gameTimer, 1000);
	}

	function focusInput() {
		inputEl.focus();
	}

	function blurInput() {
		inputEl.focus();
	}

	function handleKeydown(event: KeyboardEvent) {
		if (event.key !== 'Escape' && game === 'waiting for input') {
			startGame();
		}

		if (event.code === 'Space') {
			event.preventDefault();

			if (game === 'in progress') {
				nextWord();
			}
		}
	}

	function getWordsPerMinute() {
		const word = 5;
		return Math.floor(correctLetters / word);
	}

	function getPrecision() {
		const totalLetters = words.reduce((count, word) => count + word.length, 0);
		return Math.floor((correctLetters / totalLetters) * 100);
	}

	function getResults() {
		$wordsPerMinute = getWordsPerMinute();
		$precision = getPrecision();
	}

	function pauseGame(e: KeyboardEvent) {
		if (e.key === 'Escape') {
			e.preventDefault();
			switch (game) {
				case 'waiting for input':
					game = 'paused';
					dialog.showModal();
					break;
				case 'in progress':
					game = 'paused';
					dialog.showModal();
					break;
				case 'paused':
					if (!letterEl) game = 'waiting for input';
					else game = 'in progress';
					dialog.close();
					break;
			}
		}
	}

	onMount(() => {
		focusInput();
	});
</script>

<svelte:head>
	<title>Sveltic Gam8s: Typing Race</title>
</svelte:head>

<svelte:window on:keydown={pauseGame} />

<div class="header">
	<h2>Typing race</h2>
	<p class="timer" class:timer--active={game !== 'waiting for input' && letterEl}>
		Time:<span>{seconds}</span>
	</p>
</div>

<input
	class="sr-only"
	type="text"
	bind:this={inputEl}
	bind:value={typedLetter}
	on:input={updateGameState}
	on:keydown={handleKeydown}
	on:blur={blurInput}
/>

{#key toggleReset}
	<div class="words" bind:this={wordsEl} in:blur|local>
		{#each words as word}
			<span class="word">
				{#each word as letter}
					<span class="letter">{letter}</span>
				{/each}
			</span>
		{/each}
		<div class="caret" bind:this={caretEl} />
	</div>
{/key}

<div class="info">
	<p class="info-start" class:info-start--active={letterEl}>Let's typing for start</p>
	<p class="info-layout" bind:this={messageLayout}>Change your keyboard layout to English</p>
</div>

<Dialog bind:dialog>
	<div slot="content">
		{#if game === 'paused'}
			<h2>Game paused</h2>
		{/if}
		{#if game === 'game over'}
			<h2 style="color: var(--color-blue); text-transform: uppercase;">Your results</h2>
			<div class="results">
				<div class="result">
					<p class="title">wpm</p>
					<p class="score">{Math.trunc($wordsPerMinute)}</p>
				</div>
				<div class="result">
					<p class="title">precision</p>
					<p class="score">{Math.trunc($precision)}%</p>
				</div>
			</div>
		{/if}
	</div>

	{#if game === 'paused'}
		<button
			class="nes-btn"
			on:click={() => {
				game = 'in progress';
				dialog.close();
			}}
		>
			Continue
		</button>

		<button
			class="nes-btn"
			on:click={() => {
				game = 'waiting for input';
				dialog.close();
				resetGame();
			}}
		>
			New game
		</button>
	{:else}
		<button
			class="nes-btn"
			on:click={() => {
				game = 'waiting for input';
				dialog.close();
				resetGame();
			}}
		>
			Play&nbsp;again
		</button>
	{/if}
</Dialog>

<style>
	.words {
		--line-height: 1em;
		--lines: 5;

		position: relative;

		display: flex;
		flex-wrap: wrap;
		gap: 0.4em;

		width: 100%;
		max-height: calc(var(--line-height) * var(--lines) * 1.8);

		font-size: 1.4rem;

		overflow: hidden;
		user-select: none;
	}

	.word {
		margin-right: 0.6rem;
	}

	.letter {
		opacity: 0.5;
		transition: all 0.27s ease;
	}

	:global(.letter[data-letter='correct']) {
		opacity: 0.9;
	}

	:global(.letter[data-letter='incorrect']) {
		color: var(--color-red);
		opacity: 1;
	}

	.caret {
		position: absolute;
		top: 0;
		height: 1.6rem;
		border-right: 4px solid #209cee;
		animation: impulseCaret 1s infinite;
		transition: all 0.06s ease;
	}

	@keyframes impulseCaret {
		0% {
			opacity: 0;
		}
		50% {
			opacity: 1;
		}
		100% {
			opacity: 0;
		}
	}

	:global(.results) {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-bottom: 1rem;
		text-align: center;
	}

	:global(.result .title) {
		margin-bottom: 0.4rem;
		font-size: 1.4rem;
		text-transform: uppercase;
	}

	:global(.results .score) {
		margin-bottom: 0.6rem;
		font-size: 1.6rem;
		color: var(--color-red);
	}

	.info {
		position: relative;
	}

	.info-start,
	.info-layout {
		position: absolute;
		width: 100%;
		text-align: center;
		transition: all 0.2s ease;
	}

	.info-start {
		color: var(--color-blue);
		opacity: 1;

		&.info-start--active {
			opacity: 0;
		}
	}

	.info-layout {
		color: var(--color-red);
		opacity: 0;

		&.info-layout--active {
			opacity: 1;
		}
	}
</style>
