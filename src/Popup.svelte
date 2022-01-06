<script>
  import FakeScroll from "./FakeScroll.svelte";
</script>

<div class="bg">
  <div class="fg">
    <FakeScroll>
      <slot />
    </FakeScroll>
  </div>
</div>

<style>
  * {
    box-sizing: border-box;
  }

  .bg {
    position: fixed;
    z-index: 1000;
    top: 0;
    left: 0;
    display: flex;
    flex-direction: column;
    justify-content: center;
    width: 100vw;
    height: 100vh;
    background: rgba(0, 0, 0, 0.66);
  }

  .fg {
    position: relative;
    width: 40rem;
    max-width: 90vw;
    height: 90vh;
    max-height: 90vh;
    margin: 2rem auto;
    color: black;
    border-radius: 0.5rem;
    background: white;
    padding: 10px;
  }
</style>
