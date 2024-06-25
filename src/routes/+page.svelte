<script>
	import rings from "$lib/images/olympic-rings.png";
</script>

<svelte:head>
	<title>Sveltic Gam8s</title>
</svelte:head>

<div class="bg"></div>

<img src={rings} alt="">

<h1>Sveltic Gam8s</h1>

<nav class="menu nes-container is-rounded">
	<ul class="menu__list">
		<li class="menu__item"><a class="nes-btn" href="typing-race">Typing race</a></li>
		<li class="menu__item"><a class="nes-btn" href="match-shooting">Match shooting</a></li>
		<li class="menu__item"><a class="nes-btn is-disabled" href="hop-tower" tabindex="-1">Jump of hope</a></li>
	</ul>
</nav>

<style>
	.bg {
		position: absolute;
		inset: 0;

    background-image: url("$lib/images/olympic-stadium-pixel-art.png");
    background-repeat: no-repeat;
    background-position: center;
    background-size: cover;

		z-index: -1;
	}

	h1 {
		color: #33312c;
		margin-bottom: 2rem;
		text-align: center;
		-webkit-text-stroke: 1px #3d4247;
	}

	.menu {
		padding: 2rem 2.5rem;
		background-color: wheat;
	}

	.menu__list {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1rem;
		margin: 0;
		padding: 0;
		list-style: none;
	}

	.menu__item a {
		display: inline-block;
		text-transform: uppercase;
	}

	.menu__item a.is-disabled {
		user-select: none;
		pointer-events: none;
	}

	.menu__item a:hover,
	.menu__item a:focus {
		color: var(--color-white);
		background-color: var(--color-blue);

		&::after {
			box-shadow: inset -4px -4px #006bb3;
		}
	}
</style>
