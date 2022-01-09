<script lang="ts">
  import FakeScroll from "./FakeScroll.svelte";
  export let active = false;

  let pointerDownOnBg = false;
  function closeModal(e: Event) {
    if (pointerDownOnBg) active = false;
    pointerDownOnBg = false;
  }
</script>

{#if active}
  <div
    class="bg"
    on:pointerdown|self|stopPropagation="{() => (pointerDownOnBg = true)}"
    on:pointerup|self|stopPropagation="{closeModal}"
  >
    <div class="fg">
      <FakeScroll>
        <slot />
      </FakeScroll>
    </div>
  </div>
{/if}

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
    max-width: 90vw;
    max-height: 95vh;
    margin: 2rem auto;
    color: black;
    border-radius: 6px;
    background: white;
    padding: 10px;
  }
</style>
