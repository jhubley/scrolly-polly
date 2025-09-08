<script>
  import { onMount, onDestroy } from "svelte";
  import { tweened } from "svelte/motion";
  import { cubicOut } from "svelte/easing";
  import { derived } from "svelte/store";
  import { fade } from "svelte/transition";

  let stage = 0;

  const labels = [
    "Very serious problem",
    "Somewhat serious problem",
    "Not a serious problem",
  ];
  const initialValues = [22, 34, 43];
  const updatedValues = [22, 34, 43];
  const step3Values = [10, 26, 63];

  const initialValues2 = [0, 0, 0];
  const updatedValues2 = [18, 28, 50];
  const step3Values2 = [12, 14, 30];

  const barStores1 = initialValues.map(() =>
    tweened(0, { duration: 800, easing: cubicOut }),
  );

  const barValues1 = derived(barStores1, ($bars) => $bars);

  const barStores2 = initialValues2.map(() =>
    tweened(0, { duration: 800, easing: cubicOut }),
  );
  const barValues2 = derived(barStores2, ($bars) => $bars);

  function updateBars(stage) {
    if (stage === 1) {
      // First stage: only show first chart
      barStores1.forEach((s, i) => s.set(initialValues[i]));
      barStores2.forEach((s, i) => s.set(0)); // hide second chart
    } else if (stage === 2) {
      barStores1.forEach((s, i) => s.set(initialValues[i])); // or updatedValues[i]
      barStores2.forEach((s, i) => s.set(updatedValues2[i]));
    } else if (stage === 3) {
      barStores1.forEach((s, i) => s.set(step3Values[i]));
      barStores2.forEach((s, i) => s.set(step3Values2[i]));
    }
  }

  onMount(() => {
    updateBars(stage);
  });

  function handleScroll() {
    const y = window.scrollY;

    if (y <= 300 && stage !== 1) {
      stage = 1;
      updateBars(stage);
    } else if (y > 300 && y <= 600 && stage !== 2) {
      stage = 2;
      updateBars(stage);
    } else if (y > 600 && stage !== 3) {
      stage = 3;
      updateBars(stage);
    }
  }

  onMount(() => {
    window.addEventListener("scroll", handleScroll);
  });

  onDestroy(() => {
    window.removeEventListener("scroll", handleScroll);
  });
</script>

<div class="scroll-container">
  <!-- Intro text -->
  <div class="intro-text">
    <h2>Perception of crime</h2>
    <p>
      Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod
      tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
      veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
      commodo consequat. Duis aute irure dolor in reprehenderit in voluptate
      velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat
      cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id
      est laborum.
    </p>
  </div>

  <!-- Sticky container with persistent layout -->
  <div class="sticky-chart">
    <!-- Left panel: always present -->
    <div class="left-panel">
      {#key stage}
        {#if stage === 1}
          <div class="overlay-text" transition:fade>
            <p>
              In November 2022, roughly a quarter of people surveyed thought
              crime was a pretty big problem where they lived.
            </p>
          </div>
        {:else if stage === 2}
          <div class="overlay-text" transition:fade>
            <p>
              Nationally, perceptions of crime were higher — but still lower
              than expected.
            </p>
          </div>
        {:else if stage === 3}
          <div class="overlay-text" transition:fade>
            <p>
              By 2025, concern about crime dropped even further in both local
              and national contexts.
            </p>
          </div>
        {/if}
      {/key}
    </div>

    <!-- Right panel: bar chart -->
    <div class="bar-container">
      <div class="bar-row">
        <div class="label"></div>
        <div class="place">Local/neighborhood</div>
      </div>
      {#each labels as label, i}
        <div class="bar-row">
          <div class="label">{label}</div>
          <div class="bar" style="width: {$barValues1[i] * 4}px">
            {Math.round($barValues1[i]) + "%"}
          </div>
        </div>
      {/each}
    </div>

    {#if stage > 1}
      <div class="bar-container">
        <div class="bar-row">
          <div class="label"></div>
          <div class="place">U.S.</div>
        </div>
        {#each labels as label, i}
          <div class="bar-row">
            <div class="label">{label}</div>
            <div class="bar" style="width: {$barValues2[i] * 4}px">
              {Math.round($barValues2[i]) + "%"}
            </div>
          </div>
        {/each}
      </div>
    {/if}
  </div>

  <!-- Extra scroll space -->
  <!-- <div style="height: 300vh;"></div> -->
</div>
<div class="extra-paragraph">
  <p>
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus lacinia
    odio vitae vestibulum vestibulum. Cras venenatis euismod malesuada. Etiam at
    orci nec leo placerat sollicitudin. Integer mollis volutpat lacus, a feugiat
    tortor volutpat eu.
  </p>
</div>

<style>
  .scroll-container {
    height: 200vh;
    position: relative;
    padding: 0 1rem;
    max-width: 1000px;
    margin: 0 auto;
  }

  .intro-text {
    margin: 2rem 0;
    font-size: 1.1rem;
    line-height: 1.5;
    text-align: left;
  }

  .sticky-chart {
    position: sticky;
    top: 20px;
    width: 100%;
    height: 350px;
    z-index: 10;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    margin-bottom: 1rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 1rem;
    box-sizing: border-box;
  }

  .left-panel {
    width: 40%;
    max-width: 300px;
    box-sizing: border-box;
    flex-shrink: 0;
    min-height: 100%;
    display: flex;
    align-items: center;
  }

  .overlay-text,
  .placeholder-text {
    width: 100%;
    font-size: 1rem;
    line-height: 1.4;
    background: rgba(255, 255, 255, 0.9);
    padding: 1rem;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    box-sizing: border-box;
    color: #000;
  }

  .placeholder-text {
    opacity: 0.5;
    color: #888;
    font-style: italic;
  }

  .bar-container {
    /* display: flex;
    flex-direction: column;
    justify-content: center; */
    flex: none;
    gap: 0.5rem;
    border-radius: 8px;
    width: 435px !important;
    box-sizing: border-box;
  }

  .bar-row {
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  .label {
    width: 160px;
    font-weight: 600;
    text-align: right;
  }

  .bar {
    height: 30px;
    background-color: #3498db;
    color: white;
    display: flex;
    align-items: center;
    justify-content: flex-start; /* Change to left align */
    padding-left: 0.5rem; /* Add some padding on left instead of right */
    border-radius: 0px;
    transition: width 0.3s;
    white-space: nowrap;
  }
  .place {
    text-align: left;
  }
</style>
