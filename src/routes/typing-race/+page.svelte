<script lang="ts">
	import type { PageData } from './$types';

	export let data: PageData;

	type Game = 'Waiting for input' | 'In progress' | 'Game Over';
	type Word = string;

	let game: Game = 'Waiting for input';
	let typedLetter = '';

	let words: Word[] = 'A bird in the hand is worth two in the bush'.split(' ');
	let wordIndex = 0;
	let letterIndex = 0;
	let correctLetters = 0;

	let wordsEl: HTMLDivElement;
	let letterEl: HTMLSpanElement;
	let inputEl: HTMLInputElement;

	function updateGameState() {}

	function handleKeydown(event: KeyboardEvent) {}
</script>

<div class="game" data-game={game}></div>
