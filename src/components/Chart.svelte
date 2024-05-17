<script>
  import { onMount } from "svelte";
  import { csv } from "d3-fetch";
  import { scaleLinear } from "d3-scale";
  import { extent } from "d3-array";
  import Tooltip from "./Tooltip.svelte";

  let hoveredData;
  let innerRadius = 40;
  let selectedCirle;
  let concentricData = [];
  
  const margin = { top: 60, right: 90, bottom: 60, left: 60 };
  let width = 900;
  let height = 900;

  onMount(async () => {
    let fetchConcentricData = await csv("/summary12.csv", (data) => ({
      territory: data["Territory"],
      averageDeliveryTime: +data["Average_Delivery_Time"],
    }));
    concentricData = [...fetchConcentricData];
  });

  $: avgTimes = concentricData
    // .map((item) => item.averageDeliveryTime)
    .sort((a, b) => b.averageDeliveryTime - a.averageDeliveryTime)
    .slice(0, 50);

  $: rScale = scaleLinear()
    .domain(extent(avgTimes, (d) => d.averageDeliveryTime))
    .range([innerRadius, height / 2 - 30]);

  $: colorScale = scaleLinear()
    .domain(extent(avgTimes, (d) => d.averageDeliveryTime))
    .range(["#7ABA78", "#1A4D2E"]);
</script>

<main>
  <div class="container" bind:clientHeight={height} bind:clientWidth={width}>
    <div
      style="position: absolute; font-size: 1.475em; margin-top:0.25em; margin-top: 1.85em; left:50px; color: white;"
    >
      Top 50 Fastest Delivery Time
    </div>
    {#if hoveredData}
      <Tooltip {rScale} data={hoveredData} />
    {/if}

    <svg>
      <g transform="translate({margin.left} {margin.top})">
        {#each avgTimes as item, i}
          <circle
            r={rScale(item.averageDeliveryTime)}
            stroke={selectedCirle === item ? "#FF9800" : "white"}
            cx={(width - margin.right - margin.left) / 2}
            cy={(height - margin.top - margin.bottom) / 2}
            stroke-width="0.5"
            on:focus={(e) => e}
            fill={selectedCirle === item
              ? "#0E8388"
              : colorScale(item.averageDeliveryTime)}
            on:mouseover={() => {
              selectedCirle = item;
              hoveredData = item;
            }}
            on:mouseleave={() => {
              selectedCirle = null;
            }}
          />
        {/each}

        {#each avgTimes as item, i}
          <text
            class="legend "
            y={9.5 * i}
            x={width * 0.8}
            fill={selectedCirle === item ? "#FF9800" : "white"}
          >
            {i + 1}. {item.territory} - {item.averageDeliveryTime}
          </text>
        {/each}

        <defs>
          <linearGradient id="Gradient2" x1="0" x2="0" y1="0" y2="1">
            <stop offset="0%" stop-color="#1A4D2E" />
            <stop offset="100%" stop-color="#7ABA78" />
          </linearGradient>
        </defs>

        <rect
          x={0}
          y={height / 2 - margin.bottom}
          width="15"
          height="200"
          fill="url(#Gradient2)"
        />
        <text x="0" y="480" dx="-3" fill="white">10</text>
        <text x="0" y="230" dx="-3" fill="white">15</text>
      </g>
    </svg>
  </div>
</main>

<style>
  svg {
    background-color: #102c57;
    min-height: 100vh;
    width: 100%;
  }
  circle {
    transition: all 200ms ease;
  }

  .container {
    /* background-color: brown; */
    position: relative;
  }

  .legend {
    font-size: 0.55em;
  }
</style>
