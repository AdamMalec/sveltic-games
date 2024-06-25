<script>
	import '../app.css';

</script>

<div class="container">
  <slot />
</div>

<style>
  .container {
    display: grid;
    place-items: center;
    align-content: center;
    height: 100vh;
  }
</style>
