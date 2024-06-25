<script lang="ts">
	import Dialog from '$lib/components/Dialog.svelte';
	import { emoji } from '../../../emoji';

	type GameState = 'start' | 'playing' | 'paused' | 'win' | 'lose';

	let state: GameState = 'start';
	let size = 20;
	let grid = createGrid();
	let maxMatches = grid.length / 2;
	let selected: number[] = [];
	let matches: string[] = [];

	let timerId: number | null = null;
	let seconds = 60;

	let dialog: HTMLDialogElement;

	$: selected.length === 2 && matchCards();
	$: maxMatches === matches.length && winGame();

	function shuffle<Items>(array: Items[]) {
		return array.sort(() => Math.random() - 0.5);
	}

	function createGrid() {
		let cards = new Set<string>();
		let maxSize = size / 2;

		while (cards.size < maxSize) {
			const randomIndex = Math.floor(Math.random() * emoji.length);
			cards.add(emoji[randomIndex]);
		}

		return shuffle([...cards, ...cards]);
	}

	function selectCard(cardIndex: number) {
		selected = [...selected, cardIndex];
	}

	function matchCards() {
		const [first, second] = selected;

		if (grid[first] === grid[second]) {
			matches = [...matches, grid[first]];
		}

		setTimeout(() => (selected = []), 400);
	}

	function winGame() {
		state = 'win';
		dialog.showModal();
		resetGame();
	}

	function loseGame() {
		state = 'lose';
		dialog.showModal();
		resetGame();
	}

	function startGameTimer() {
		function countdown() {
			state !== 'paused' && (seconds -= 1);
		}

		timerId = setInterval(countdown, 1000);
	}

	$: if (state === 'playing') {
		//	in case you pause the game
		!timerId && startGameTimer();
	}

	$: seconds === 0 && loseGame();

	function resetGame() {
		timerId && clearInterval(timerId);
		timerId = null;
		seconds = 60;
		grid = createGrid();
		maxMatches = grid.length / 2;
		selected = [];
		matches = [];
	}

	function pauseGame(e: KeyboardEvent) {
		if (e.key === 'Escape') {
			e.preventDefault();
			switch (state) {
				case 'start':
					state = 'paused';
					dialog.showModal();
					break;
				case 'playing':
					state = 'paused';
					dialog.showModal();
					break;
				case 'paused':
					state = 'playing';
					dialog.close();
					break;
				case 'win':
					state = 'playing';
					dialog.close();
					break;
				case 'lose':
					state = 'playing';
					dialog.close();
					break;
			}
		}
	}
</script>

<svelte:head>
	<title>Sveltic Gam8s: Match shooting</title>
</svelte:head>

<svelte:window on:keydown={pauseGame} />

<div class="header">
	<h2>Match shooting</h2>
	<p class="timer" class:timer--active={state === 'playing' || (state === 'paused' && seconds !== 60)}>
		Time:<span>{seconds}</span>
	</p>
</div>

<div class="cards">
	{#each grid as card, cardIndex}
		{@const isSelected = selected.includes(cardIndex)}
		{@const isMatched = matches.includes(card)}
		{@const isSelectedOrMatch = isSelected || isMatched}
		<button
			class="card nes-container is-rounded"
			class:selected={isSelected}
			class:flip={isSelectedOrMatch}
			on:click={() => selectCard(cardIndex)}
			on:click={() => {
				if (state === 'start') state = 'playing';
			}}
			disabled={isSelectedOrMatch}
		>
			<div class="back" class:matched={isMatched}>{card}</div>
		</button>
	{/each}
</div>

<Dialog bind:dialog>
	<h2 slot="content">
		{#if state === 'win'}You Win&nbsp;<i class="nes-icon trophy is-medium"></i>{/if}
		{#if state === 'lose'}You Lose{/if}
		{#if state === 'paused'}Game Paused{/if}
	</h2>

	{#if state === 'paused'}
		<button
			class="nes-btn"
			on:click={() => {
				state = 'playing';
				dialog.close();
			}}
		>
			Continue
		</button>

		<button
			class="nes-btn"
			on:click={() => {
				state = 'start';
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
				state = 'start';
				dialog.close();
				resetGame();
			}}
		>
			Play&nbsp;again
		</button>
	{/if}
</Dialog>

<style>
	.header h2 {
		text-align: center;
	}

	.cards {
		align-self: start;
		display: grid;
		grid-template-columns: repeat(5, 20vmin);
		justify-content: center;
		gap: 0.4rem;
	}

	.card {
		aspect-ratio: 1/1;
		font-size: 3.6rem;
		background-color: var(--color-bg-2);
		transition: rotate 0.3s ease-out;
		transform-style: preserve-3d;
		user-select: none;

		&.selected {
			background-color: var(--color-green);
		}

		&.flip {
			rotate: y 180deg;
			pointer-events: none;
		}

		& .back {
			position: absolute;
			inset: 0;
			display: grid;
			place-content: center;
			backface-visibility: hidden;
			rotate: y 180deg;
		}

		& .matched {
			transition: opacity 0.3s ease-out;
			opacity: 0.4;
			cursor: pointer;
		}

		&:disabled {
			color: unset;
		}
	}
</style>
