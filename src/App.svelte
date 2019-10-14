<script>
  import ImprovedSlider from "./ImprovedSlider.svelte";
  import Slider from "./Slider.svelte";

  const numberOfColumns = 4;
  let toggle = false;
</script>

<style>
  .outerWidth {
    /* margin-left: 50%; */
  }

  .parentWidth {
    /* padding-left: 20%; */
  }
</style>

<button on:click={e => (toggle = !toggle)}>Toggle</button>

{#if toggle}
  <div class="parentWidth">
    <div class="outerWidth">
      <!-- <ImprovedSlider
      columnWidthInPercent={[25, 25, 25, 10, 10, 5]}
      on:columnWidthChange={e => console.log(e.detail.columnsWidths)} /> -->
      <div class="container">
        <Slider gapBetweenThumbsInPercent={[0, 50, 50, 0]} />
      </div>
    </div>
  </div>
{/if}
