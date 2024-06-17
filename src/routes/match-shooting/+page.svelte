<script lang="ts">
	import { emoji } from '../../emoji';

	type GameState = 'start' | 'playing' | 'paused' | 'won' | 'lost';

	let state: GameState = 'start';
	let size = 20;
	let grid = createGrid();
	let maxMatches = grid.length / 2;
	let selected: number[] = [];
	let matches: string[] = [];

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
</script>

{#if state === 'start'}
	<button on:click={() => (state = 'playing')}> Play </button>
{/if}

{#if state === 'playing'}
	<div class="cards">
		{#each grid as card, cardIndex}
			<button class="card">
				<div>{card}</div>
			</button>
		{/each}
	</div>
{/if}

{#if state === 'lost'}
	<h2>You lose 💩</h2>
	<button on:click={() => state = 'playing'}>
    Play again
  </button>
{/if}

{#if state === 'won'}
	<h2>You win! 🍾</h2>
	<button on:click={() => state = 'playing'}>
    Play again
  </button>
{/if}

<style>
	.cards {
		display: grid;
		grid-template-columns: repeat(5, 1fr);
		gap: 0.4rem;
	}

	.card {
		height: 140px;
		width: 140px;
		font-size: 4rem;
		background-color: var(--bg-2);

		&.selected {
			border: 4px solid var(--border);
		}
	}
</style>
