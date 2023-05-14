<script lang="ts">
	import type { PageData } from './$types';
	import { tweened } from 'svelte/motion';

	export let data: PageData;

	type Game = 'waiting for input' | 'in progress' | 'game over';
	type Word = string;

	let game: Game = 'waiting for input';
	let seconds = 60;
	let typedLetter = '';

	let words: Word[] =
		'There were ninety-seven New York advertising men in the hotel, and, the way they were monopolizing the long-distance lines, the girl in 507 had to wait from noon till almost two-thirty to get her call through. She used the time, though. She read an article in a women\'s pocket-size magazine, called "Sex Is Fun- or Hell." She washed her comb and brush. She took the spot out of the skirt of her beige suit. She moved the button on her Saks blouse. She tweezed out two freshly surfaced hairs in her mole. When the operator finally rang her room, she was sitting on the window seat and had almost finished putting lacquer on the nails of her left hand. She was a girl who for a ringing phone dropped exactly nothing. She looked as if her phone had been ringing continually ever since she had reached puberty.'.split(
			' '
		);
	let wordIndex = 0;
	let letterIndex = 0;
	let correctLetters = 0;

	let wordsPerMinute = tweened(0, { delay: 300, duration: 1000 });
	let precision = tweened(0, { delay: 300, duration: 1000 });

	let wordsEl: HTMLDivElement;
	let letterEl: HTMLSpanElement;
	let inputEl: HTMLInputElement;
	let caretEl: HTMLDivElement;

	function updateGameState() {
		setLetter();
		checkLetter();
		nextLetter();
		resetLetter();
		updateLine();
		moveCaret();
	}

	function setLetter() {
		const isWordCompleted = letterIndex > words[wordIndex].length - 1;

		if (!isWordCompleted) {
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
		const isOneLetterWord = words[wordIndex].length === 1;
		const isNotFirstLetter = letterIndex !== 0;

		if (isOneLetterWord || isNotFirstLetter) {
			wordIndex += 1;
			letterIndex = 0;
			increaseScore();
			moveCaret();
		}
	}

	function moveCaret() {
		const offset = 2;
		const { offsetTop, offsetLeft, offsetWidth } = letterEl;

		caretEl.style.top = `${offsetTop}px`;
		caretEl.style.left = `${offsetLeft + offsetWidth - offset}px`;
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
				setGameState('game over');
				getResults();
			}
		}

		const interval = setInterval(gameTimer, 1000);
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
</script>

<div class="game" data-game={game}>
	<input
		class="input"
		type="text"
		bind:this={inputEl}
		bind:value={typedLetter}
		on:input={updateGameState}
		on:keydown={handleKeydown}
	/>

	<div class="time">{seconds}</div>

	<div class="words" bind:this={wordsEl}>
		{#each words as word}
			<span class="word">
				{#each word as letter}
					<span class="letter">{letter}</span>
				{/each}
			</span>
		{/each}
		<div class="caret" bind:this={caretEl} />
	</div>
</div>

{#if game === 'game over'}
	<section>
		<dialog class="nes-dialog" id="dialog-default" open>
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
					<button class="nes-btn is-primary">Restart</button>
					<button class="nes-btn">Main menu</button>
				</menu>
			</form>
		</dialog>
	</section>
{/if}

<style>
	.game {
		position: relative;
	}

	.time {
		position: absolute;
		top: -3rem;
		font-size: 1.6rem;
		color: #e76e55;
		opacity: 0;
		transition: all 0.2 ease;
	}

	:global(.game[data-game='in progress'] .time) {
		opacity: 1;
	}

	.words {
		--line-height: 1em;
		--lines: 3;

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
		border-right: 4px solid #92cc41;
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

	:global(.nes-dialog) {
		min-width: 320px;
		width: 42%;
	}

	.results {
		display: flex;
		flex-direction: column;
		align-items: center;
		text-align: center;
	}

	:global(.dialog-menu) {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 0;
	}

	:global(.dialog-menu button) {
		width: 180px;
	}

	:global(.dialog-menu button:first-of-type) {
		margin-bottom: 1rem;
	}
</style>
