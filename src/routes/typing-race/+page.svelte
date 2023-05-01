<script lang="ts">
	import type { PageData } from './$types';

	export let data: PageData;

	type Game = 'waiting for input' | 'in progress' | 'game over';
	type Word = string;

	let game: Game = 'waiting for input';
	let typedLetter = '';

	let words: Word[] = 'A bird in the hand is worth two in the bush'.split(' ');
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
		max-height: calc(var(--line-height) * var(--lines) * 1.34);

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
