<script lang="ts">
	import type { PageData } from './$types';

	export let data: PageData;

	type Game = 'waiting for input' | 'in progress' | 'game over';
	type Word = string;

	let game: Game = 'waiting for input';
	let typedLetter = '';

	let words: Word[] = "There were ninety-seven New York advertising men in the hotel, and, the way they were monopolizing the long-distance lines, the girl in 507 had to wait from noon till almost two-thirty to get her call through. She used the time, though. She read an article in a women's pocket-size magazine, called \"Sex Is Fun- or Hell.\" She washed her comb and brush. She took the spot out of the skirt of her beige suit. She moved the button on her Saks blouse. She tweezed out two freshly surfaced hairs in her mole. When the operator finally rang her room, she was sitting on the window seat and had almost finished putting lacquer on the nails of her left hand. She was a girl who for a ringing phone dropped exactly nothing. She looked as if her phone had been ringing continually ever since she had reached puberty.".split(' ');
	let wordIndex = 0;
	let letterIndex = 0;
	let correctLetters = 0;

	let wordsEl: HTMLDivElement;
	let letterEl: HTMLSpanElement;
	let inputEl: HTMLInputElement;

	function updateGameState() {
		setLetter();
		checkLetter();
		nextLetter();
		resetLetter();
		updateLine();
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
			wordEl.scrollIntoView({block: 'center'});
		}
	}

	function nextWord() {
		const isOneLetterWord = words[wordIndex].length === 1;
		const isNotFirstLetter = letterIndex !== 0;

		if (isOneLetterWord || isNotFirstLetter) {
			wordIndex += 1;
			letterIndex = 0;
			increaseScore();
		}
	}

	function startGame() {
		setGameState('in progress');
	}

	function setGameState(state: Game) {
		game = state;
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

	<div class="words" bind:this={wordsEl}>
		{#each words as word}
			<span class="word">
				{#each word as letter}
					<span class="letter">{letter}</span>
				{/each}
			</span>
		{/each}
	</div>
</div>

<style>
	.words {
		--line-height: 1em;
		--lines: 3;

		display: flex;
		flex-wrap: wrap;
		gap: 0.4em;

		width: 100%;
		max-height: calc(var(--line-height) * var(--lines) * 1.8);

		font-size: 1.4rem;

		overflow: hidden;
		user-select: none;
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
</style>
