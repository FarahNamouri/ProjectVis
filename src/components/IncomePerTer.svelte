
 <script>
    import Papa from 'papaparse';
    import { onMount } from 'svelte';

    let data = [];
    let incomePerTerritory = {};

    onMount(() => {
      fetch('/orders.csv').then(response => {
        Papa.parse(response.body, {
          header: true,
          download: true,
          step: (result) => {
            const row = result.data;
            const territory = row.Territory;
            const income = parseFloat(row.CartPriceInCp);
            if (territory in incomePerTerritory) {
              incomePerTerritory[territory] += income;
            } else {
              incomePerTerritory[territory] = income;
            }
            updateData();
          },
          complete: () => {
            console.log("Finished parsing");
          }
        });
      });
    });

    function updateData() {
        data = Object.entries(incomePerTerritory).map(([territory, income]) => ({ territory, income }));
        data.sort((a, b) => b.income - a.income);
    }
</script>

<style>
    .bar {
      height: 20px;
      margin: 5px 0;
      color: white;
      display: flex;
      align-items: center;
      padding-left: 10px;
    }
</style>

{#each data as { territory, income }}
    <div class="bar" style="width: {income / data[0].income * 100}%; background-color: hsl({Math.random() * 360}, 70%, 50%)">
      {territory}: ${income.toFixed(2)}
    </div>
{/each}
