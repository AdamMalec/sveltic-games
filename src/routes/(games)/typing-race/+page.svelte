<script lang="ts">
	import { onMount } from 'svelte';
	import { tweened } from 'svelte/motion';
	import { blur } from 'svelte/transition';

	type Game = 'waiting for input' | 'in progress' | 'game over';
	type Word = string;

	let game: Game = 'waiting for input';
	let seconds = 10;
	let typedLetter = '';

	let words: Word[] =
		`If you really want to hear about it, the first thing you’ll probably want to know is where I was born, an what my lousy childhood was like, and how my parents were occupied and all before they had me, and all that David Copperfield kind of crap, but I don’t feel like going into it, if you want to know the truth. In the first place, that stuff bores me, and in the second place, my parents would have about two hemorrhages apiece if I told anything pretty personal about them. They’re quite touchy about anything like that, especially my father. They’re nice and all—I’m not saying that—but they’re also touchy as hell. Besides, I’m not going to tell you my whole goddam autobiography or anything. I’ll just tell you about this madman stuff that happened to me around last Christmas just before I got pretty run-down and had to come out here and take it easy. I mean that’s all I told D.B. about, and he’s my brother and all. He’s in Hollywood. That isn’t too far from this crumby place, and he comes over and visits me practically every week end. He’s going to drive me home when I go home next month maybe. He just got a Jaguar. One of those little English jobs that can do around two hundred miles an hour. It cost him damn near four thousand bucks. He’s got a lot of dough, now. He didn’t use to. He used to be just a regular writer, when he was home. He wrote this terrific book of short stories, The Secret Goldfish, in case you never heard of him. The best one in it was “The Secret Goldfish.” It was about this little kid that wouldn’t let anybody look at his goldfish because he’d bought it with his own money. It killed me. Now he’s out in Hollywood, D.B., being a prostitute. If there’s one thing I hate, it’s the movies. Don’t even mention them to me.`.split(
			' '
		);
	let wordIndex = 0;
	let letterIndex = 0;
	let correctLetters = 0;
	let toggleReset = false;

	let wordsPerMinute = tweened(0, { delay: 300, duration: 1000 });
	let precision = tweened(0, { delay: 300, duration: 1000 });

	let wordsEl: HTMLDivElement;
	let letterEl: HTMLSpanElement;
	let inputEl: HTMLInputElement;
	let caretEl: HTMLDivElement;

	const isWordCompleted = () => letterIndex > words[wordIndex].length - 1;
	const isOneLetterWord = () => words[wordIndex].length === 1;

	function resetGame() {
		inputEl.disabled = false;
		toggleReset = !toggleReset;

		setGameState('waiting for input');
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

		if (typedLetter === currentLetter) {
			letterEl.dataset.letter = 'correct';
			increaseScore();
		}

		if (typedLetter !== currentLetter) {
			letterEl.dataset.letter = 'incorrect';
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
		const { offsetTop, offsetLeft, offsetWidth } = letterEl;

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
			if (seconds > 0) {
				seconds -= 1;
			}

			if (game === 'waiting for input' || seconds === 0) {
				clearInterval(interval);
			}

			if (seconds === 0) {
				inputEl.disabled = true;
				setGameState('game over');
				getResults();
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
		if (game === 'waiting for input') {
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

	onMount(() => {
		focusInput();
	});
</script>

<svelte:head>
	<title>Sveltic Gam8s: Typing Race</title>
</svelte:head>

<div class="header">
	<h2>Typing race</h2>
	<p class="timer" class:timer--active={game === 'in progress'}>Time:<span>{seconds}</span></p>
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

<button class="nes-btn is-primary restart-btn" on:click={resetGame}>Restart</button>

{#if game === 'game over'}
	<dialog class="nes-dialog" open>
		<form method="dialog">
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

			<menu class="dialog-menu">
				<button class="nes-btn is-primary" on:click={resetGame}>Try again</button>
				<a href="/" class="nes-btn">Main menu</a>
			</menu>
		</form>
	</dialog>
{/if}

<style>
	.timer {
		opacity: 0;
		transition: all 0.3s ease;
	}

	.timer--active {
		opacity: 1;
	}

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
		color: #e76e55;
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

	.restart-btn {
		align-self: start;
		display: block;
		margin: 6rem auto 0.4rem auto;
	}

	:global(.nes-dialog) {
		min-width: 320px;
		width: 38%;
	}

	:global(.results) {
		display: flex;
		flex-direction: column;
		align-items: center;
		height: 240px;
		text-align: center;
	}

	:global(.result) {
		margin-bottom: 0.6rem;
	}

	:global(.result .title) {
		margin-bottom: 0.4rem;
		font-size: 1.4rem;
		text-transform: uppercase;
	}

	:global(.results .score) {
		font-size: 1.6rem;
		color: #e76e55;
	}

	:global(.dialog-menu) {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1rem;
		padding: 0;
	}

	:global(.dialog-menu button) {
		width: 180px;
	}
</style>
