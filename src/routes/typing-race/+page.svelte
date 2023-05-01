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

	function updateGameState() {}

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
