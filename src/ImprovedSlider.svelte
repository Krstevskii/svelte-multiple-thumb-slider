<script>
  import { onMount } from "svelte";
  import { createEventDispatcher } from "svelte";
  export let columnWidthInPercent = undefined;
  let sliderThumbsOffsets = [];
  export let minimumGap = undefined;
  let gap = 0;

  const dispatch = createEventDispatcher();

  let selectedThumb = undefined;
  // DOM
  let sliderTrackOffset = undefined;
  let sliderTrackWidth = undefined;

  $: selectedThumb, rePositionThumb();
  $: sliderThumbsOffsets, calculateColumnsWidths();

  function calculateColumnsWidths() {
    let prevOffset = 0;
    let gapPercentage = sliderThumbsOffsets.map(columnOffset => {
      const offsetDifference = columnOffset.offset - prevOffset;
      prevOffset = columnOffset.offset;
      const result = (offsetDifference / sliderTrackWidth) * 100;
      return Math.round(result * 100) / 100;
    });

    const result = (1 - prevOffset / sliderTrackWidth) * 100;
    gapPercentage = [...gapPercentage, Math.round(result * 100) / 100];

    dispatch("columnWidthChange", {
      columnsWidths: gapPercentage
    });
  }

  function rePositionThumb() {
    if (selectedThumb) {
      const id = parseInt(selectedThumb.id);
      if (sliderThumbsOffsets[id - 1]) {
        if (selectedThumb.offset <= sliderThumbsOffsets[id - 1].offset + gap) {
          sliderThumbsOffsets[id].offset =
            sliderThumbsOffsets[id - 1].offset + gap;
        }
      } else {
        if (selectedThumb.offset <= gap) {
          sliderThumbsOffsets[id].offset = gap;
        }
      }

      if (sliderThumbsOffsets[id + 1]) {
        if (selectedThumb.offset >= sliderThumbsOffsets[id + 1].offset - gap) {
          sliderThumbsOffsets[id].offset =
            sliderThumbsOffsets[id + 1].offset - gap;
        }
      } else {
        if (selectedThumb.offset >= sliderTrackWidth - gap) {
          sliderThumbsOffsets[id].offset = sliderTrackWidth - gap;
        }
      }
    }
  }

  function getThumbTarget(event) {
    let { parentNode } = event.target;

    while (parentNode.classList[0] !== "thumb") {
      parentNode = parentNode.parentNode;
    }

    if (parentNode === null) {
      return 0;
    }

    selectedThumb = {
      id: parentNode.id,
      offset: parentNode.offsetLeft
    };

    document.onmousemove = slideThumbHorizontally;
    document.onmouseup = close;
  }

  function close() {
    document.onmousemove = null;
    document.onmouseup = null;
  }

  function slideThumbHorizontally(event) {
    const { clientX } = event;

    const { id } = selectedThumb;

    selectedThumb.offset = clientX - sliderTrackOffset;
    sliderThumbsOffsets[id].offset = clientX - sliderTrackOffset;

    console.log(
      "mouse offset: ",
      clientX,
      "thumb offset: ",
      selectedThumb.offset
    );
  }

  onMount(() => {
    const sliderTrack = document.querySelector(".slider-track");

    //
    sliderTrackWidth = sliderTrack.clientWidth;
    sliderTrackOffset = sliderTrack.offsetLeft;

    if (minimumGap) {
      gap = (sliderTrackWidth * minimumGap) / 100;
    } else {
      gap = 7;
    }

    const sliderWidthPercentage = columnWidthInPercent.reduce(
      (acc, currentValue) => acc + currentValue
    );
    if (sliderWidthPercentage === 100) {
      columnWidthInPercent
        .splice(0, columnWidthInPercent.length - 1)
        .forEach((columnWidth, index, self) => {
          let previousThumbOffset = 0;
          if (sliderThumbsOffsets[index - 1]) {
            previousThumbOffset = sliderThumbsOffsets[index - 1].offset;
          } else {
            previousThumbOffset = 0;
          }

          sliderThumbsOffsets[index] = {
            id: index,
            offset:
              previousThumbOffset +
              (sliderTrack.clientWidth * columnWidth) / 100
          };
        });
    }
  });
</script>

<style>
  .multiple-thumbs-slider {
    margin-top: 5%;
  }
  .slider-track {
    width: 100%;
    height: 4px;
    background-color: grey;
    border-radius: 2px;
  }

  .thumb {
    margin-top: -26px;
    height: 26px;
    width: 13px;
    position: absolute;
    display: inline;
    opacity: 0.6;
  }

  .thumb-neck {
    height: 12.5px;
    width: 100%;
    background-color: black;
  }

  .thumb-point {
    width: 0;
    height: 0;
    border-left: 6.5px solid transparent;
    border-right: 6.5px solid transparent;
    border-top: 13px solid black;
  }
  .thumb-border {
    margin: 0 auto;
    width: 1px;
    height: 100%;
    background-color: turquoise;
  }
</style>

<div class="multiple-thumbs-slider">
  <div class="slider-track">
    <div class="slider-thumbs">
      {#each sliderThumbsOffsets as thumb, i}
        <div
          class="thumb"
          style="left: {thumb.offset}px;"
          id={i}
          on:mousedown={getThumbTarget}>
          <div class="thumb-neck">
            <div class="thumb-border" />
          </div>
          <div class="thumb-point" />
        </div>
      {/each}
    </div>
  </div>
</div>
