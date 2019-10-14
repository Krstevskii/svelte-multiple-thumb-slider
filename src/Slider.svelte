<script>
  import { onMount, createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  // input properties
  export let gapBetweenThumbsInPercent = undefined;
  export let minimumGap = undefined;

  // model properties
  let thumbsOffsets = [];
  let selectedThumb = undefined;

  // DOM nodes
  let bodyNodeWidth = undefined;
  let multipleThumbSliderNode = undefined;
  let leftOffset = undefined;
  let multipleThumbSliderNodeOffsetToLeftWindow = undefined;

  $: selectedThumb, rePositionThumb();
  $: thumbsOffsets, calculateGapWidthInPercent();

  function calculateGapWidthInPercent() {
    if (leftOffset) {
      let prevOffset = leftOffset;
      let gapPercentage = thumbsOffsets.map(columnOffset => {
        const offsetDifference = columnOffset.offset - prevOffset;
        prevOffset = columnOffset.offset;
        const result =
          (offsetDifference / multipleThumbSliderNode.clientWidth) * 100;

        return Math.round(result);
      });
      const offsetDifference =
        ((multipleThumbSliderNode.clientWidth - prevOffset + leftOffset) /
          multipleThumbSliderNode.clientWidth) *
        100;
      gapPercentage = [...gapPercentage, Math.round(offsetDifference)];

      console.log("gapPercentage", gapPercentage);

      const difference =
        100 - gapPercentage.reduce((acc, currentValue) => acc + currentValue);

      if (difference !== 0) {
        gapPercentage[selectedThumb.id] =
          gapPercentage[selectedThumb.id] + difference;
      }

      dispatchGapBetweenThumbs(gapPercentage);
    }
  }

  function dispatchGapBetweenThumbs(gapPercentage) {
    dispatch("changeInThumbsGap", {
      payload: gapPercentage,
      hasError: false
    });
  }

  function rePositionThumb() {
    if (selectedThumb) {
      const id = parseInt(selectedThumb.id);

      if (thumbsOffsets[id - 1]) {
        if (selectedThumb.offset <= thumbsOffsets[id - 1].offset + minimumGap) {
          thumbsOffsets[id].offset = thumbsOffsets[id - 1].offset + minimumGap;
        }
      } else {
        if (selectedThumb.offset <= leftOffset) {
          thumbsOffsets[id].offset = leftOffset;
        }
      }

      if (thumbsOffsets[id + 1]) {
        if (selectedThumb.offset >= thumbsOffsets[id + 1].offset - minimumGap) {
          thumbsOffsets[id].offset = thumbsOffsets[id + 1].offset - minimumGap;
        }
      } else {
        if (
          selectedThumb.offset >=
          leftOffset + multipleThumbSliderNode.clientWidth
        ) {
          thumbsOffsets[id].offset =
            leftOffset + multipleThumbSliderNode.clientWidth;
        }
      }
    }
  }

  function getSelectedThumb(event) {
    let { parentNode: parent } = event.target;

    const thumbClass = "thumb";

    while (parent.classList[0] !== thumbClass) {
      parent = parent.parentNode;
    }

    if (parent === null) {
      alert("Slider thumb not found");
    }

    const { id } = parent;

    selectedThumb = {
      ...thumbsOffsets[id]
    };

    document.onmousemove = slideThumbHorizontally;
    document.onmouseup = close;
  }

  function slideThumbHorizontally(event) {
    const { clientX } = event;
    const { id } = selectedThumb;

    selectedThumb.offset =
      clientX -
      multipleThumbSliderNodeOffsetToLeftWindow +
      multipleThumbSliderNode.offsetLeft -
      minimumGap;
    thumbsOffsets[id].offset =
      clientX -
      multipleThumbSliderNodeOffsetToLeftWindow +
      multipleThumbSliderNode.offsetLeft -
      minimumGap;
  }

  function close() {
    document.onmousemove = null;
    document.onmouseup = null;
  }

  onMount(() => {
    bodyNodeWidth = document.body.clientWidth;
    multipleThumbSliderNode = document.querySelector(".multiple-thumbs-slider");
    multipleThumbSliderNodeOffsetToLeftWindow = multipleThumbSliderNode.getBoundingClientRect()
      .left;

    if (minimumGap) {
      minimumGap = (multipleThumbSliderNode.clientWidth * minimumGap) / 100;
    } else {
      minimumGap = 7;
    }

    leftOffset = multipleThumbSliderNode.offsetLeft - minimumGap;
    let previousThumbOffset = 0;

    const checkInitialGapPercentageSum = gapBetweenThumbsInPercent.reduce(
      (acc, currentValue) => acc + currentValue
    );

    if (checkInitialGapPercentageSum !== 100) {
      return;
    }

    gapBetweenThumbsInPercent
      .splice(0, gapBetweenThumbsInPercent.length - 1)
      .forEach((thumbGap, index) => {
        if (thumbsOffsets[index - 1]) {
          previousThumbOffset =
            previousThumbOffset +
            (multipleThumbSliderNode.clientWidth * thumbGap) / 100;
        } else {
          previousThumbOffset =
            (multipleThumbSliderNode.clientWidth * thumbGap) / 100;
        }
        thumbsOffsets[index] = {
          id: index,
          offset: previousThumbOffset + leftOffset
        };
      });
  });
</script>

<style>
  .multiple-thumbs-slider {
    margin-top: 7%;
    margin-bottom: 4%;
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
    background-color: #385e98;
  }
</style>

<div
  class="multiple-thumbs-slider"
  on:click={e => console.log('MULTIPLE: ', e.target.offsetLeft)}>
  <div class="slider-track">
    <div class="slider-thumbs">
      {#each thumbsOffsets as thumb, i}
        <div
          class="thumb"
          style="left: {thumb.offset}px;"
          id={i}
          on:mousedown={getSelectedThumb}
          on:click={e => console.log(e.target.parentNode, e.clientX)}>
          <div class="thumb-neck">
            <div class="thumb-border" />
          </div>
          <div class="thumb-point" />
        </div>
      {/each}
    </div>
  </div>
</div>

<svelte:window on:mousedown={e => e.clientX} />
