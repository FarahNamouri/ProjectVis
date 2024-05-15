<script>
  import { onMount, onDestroy } from "svelte";
  import Papa from "papaparse";
  import Chart from "chart.js/auto";
  import "chartjs-plugin-zoom";

  let chartTopLeft = null;
  let chartTopRight = null;
  let chartBottom = null;
  let topLeftContainer, topRightContainer, bottomContainer;

  function processData(data) {
    const dataByTerritory = {};
    data.forEach((row) => {
      if (!dataByTerritory[row.Territory]) {
        dataByTerritory[row.Territory] = {
          "Private Buyer": 0,
          "No Key Account": 0,
          "Key Account": 0,
        };
      }
      if (row.AccountType in dataByTerritory[row.Territory]) {
        dataByTerritory[row.Territory][row.AccountType]++;
      }
    });
    return dataByTerritory;
  }

  function sortAndSliceData(dataByTerritory, sliceCount, ascending = false) {
    const sorted = Object.keys(dataByTerritory)
      .map((territory) => ({
        name: territory,
        total:
          dataByTerritory[territory]["Private Buyer"] +
          dataByTerritory[territory]["No Key Account"] +
          dataByTerritory[territory]["Key Account"],
      }))
      .sort((a, b) => (ascending ? a.total - b.total : b.total - a.total))
      .slice(0, sliceCount)
      .map((item) => item.name);

    return sorted;
  }

  function getRandomTerritories(dataByTerritory, count) {
    const keys = Object.keys(dataByTerritory);
    const randomKeys = [];
    while (randomKeys.length < count) {
      const randomKey = keys[Math.floor(Math.random() * keys.length)];
      if (!randomKeys.includes(randomKey)) {
        randomKeys.push(randomKey);
      }
    }
    return randomKeys;
  }

  function createChart(
    container,
    territories,
    dataByTerritory,
    xAxisLabel,
    yAxisLabel
  ) {
    const datasets = [
      {
        label: "Private Buyer",
        data: territories.map((t) => dataByTerritory[t]["Private Buyer"]),
        backgroundColor: "rgba(102, 187, 106, 0.8)",
      },
      {
        label: "No Key Account",
        data: territories.map((t) => dataByTerritory[t]["No Key Account"]),
        backgroundColor: "rgba(66, 135, 245, 0.8)",
      },
      {
        label: "Key Account",
        data: territories.map((t) => dataByTerritory[t]["Key Account"]),
        backgroundColor: "rgba(240, 98, 146, 0.8)",
      },
    ];

    const ctx = container.getContext("2d");
    return new Chart(ctx, {
      type: "bar",
      data: {
        labels: territories,
        datasets: datasets,
      },
      options: {
        hover: {
          mode: "nearest",
          intersect: true,
          animationDuration: 400, 
          onHover: (event, chartElement) => {
            event.native.target.style.cursor = chartElement[0]
              ? "pointer"
              : "default";
          },
        },
        scales: {
          x: {
            stacked: false,
            title: {
              display: true,
              text: xAxisLabel,
              font: {
                size: 16,
              },
            },
          },
          y: {
            stacked: false,
            beginAtZero: true,
            title: {
              display: true,
              text: yAxisLabel,
              font: {
                size: 16,
              },
            },
          },
        },
      },
    });
  }

  onMount(async () => {
    const response = await fetch("/customers.csv");
    const csvData = await response.text();

    Papa.parse(csvData, {
      header: true,
      complete: (results) => {
        const dataByTerritory = processData(results.data);
        const topTerritories = sortAndSliceData(dataByTerritory, 5);
        const leastTerritories = sortAndSliceData(dataByTerritory, 5, true);
        const randomTerritories = getRandomTerritories(dataByTerritory, 5);

        chartTopLeft = createChart(
          topLeftContainer,
          topTerritories,
          dataByTerritory,
          "Top 5 Terrirtories",
          "Number of Accounts"
        );
        chartTopRight = createChart(
          topRightContainer,
          leastTerritories,
          dataByTerritory,
          "Least 5 Territories",
          "Number of Accounts"
        );
        chartBottom = createChart(
          bottomContainer,
          randomTerritories,
          dataByTerritory,
          "Random 5 Territories",
          "Number of Accounts"
        );
      },
    });
  });

  onDestroy(() => {
    if (chartTopLeft) chartTopLeft.destroy();
    if (chartTopRight) chartTopRight.destroy();
    if (chartBottom) chartBottom.destroy();
  });
</script>



<svelte:head>
  <style>
    .dashboard {
      display: grid;
      grid-template-columns: 1fr; 
      grid-template-rows: auto auto auto; 
      gap: 20px;
      max-width: 1000px;
      margin: auto;
    }
    .chart-container {
      width: 100%;
      height: 250px;
    }
    canvas {
      width: 100%;
      height: 100%;
    }
    .full-width {
      grid-column: 1 / -1; 
    }
  </style>
</svelte:head>


<!-- 
<div class="dashboard">
  <canvas bind:this={bottomContainer} class="chart-container full-width"
  ></canvas>
</div> -->



<div class="dashboard">
  <canvas bind:this={topLeftContainer} class="chart-container"></canvas>
  <canvas bind:this={topRightContainer} class="chart-container"></canvas>
  <canvas bind:this={bottomContainer} class="chart-container full-width"></canvas>
</div>

