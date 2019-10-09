<script>
  import ImprovedSlider from "./ImprovedSlider.svelte";

  const numberOfColumns = 4;
</script>

<style>
</style>

<div class="outerWidth">
  <ImprovedSlider
    columnWidthInPercent={[25, 25, 25, 25]}
    on:columnWidthChange={e => console.log(e.detail.columnsWidths)} />
</div>
