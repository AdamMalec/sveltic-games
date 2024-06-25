<script lang="ts">
	import { goto } from '$app/navigation';

	export let dialog;
</script>

<dialog class="nes-dialog is-rounded" bind:this={dialog}>
	<slot name="content" />
	<menu class="dialog-menu">
		<slot />
		<button class="nes-btn" on:click={() => goto('/')}> Main&nbsp;menu </button>
	</menu>
</dialog>

<style>
	dialog {
		min-width: 340px;
	}

	:global(dialog h2) {
		margin-bottom: 1.6rem;
		text-align: center;
	}

	.dialog-menu {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1rem;
		padding: 0;
	}

	:global(.dialog-menu button) {
		width: 180px;
	}

	@supports (color: hsl(from white h s l)) {
		:global(dialog[open]::backdrop) {
			background: linear-gradient(
					45deg,
					hsl(from var(--color-green) h s l / 0.5) 0%,
					hsl(from var(--color-blue) h s l / 0.5) 50%,
					hsl(from var(--color-red) h s l / 0.5) 100%
				), url('$lib/images/grFilter.svg');
		}
	}
</style>
