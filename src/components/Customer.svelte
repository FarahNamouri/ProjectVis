
   <script>
    import { onMount } from 'svelte';
    import Papa from 'papaparse';
    import { Pie } from 'svelte-chartjs';
  
    let data = {
      labels: [],
      datasets: [{
        data: [],
        backgroundColor: [],
      }],
    };
  
    onMount(async () => {
      const response = await fetch('/customers.csv');
      const csvData = await response.text();
  
      Papa.parse(csvData, {
        header: true,
        complete: (results) => {
          const countMap = new Map();
  
          results.data.forEach((row) => {
            const key = `${row.AccountName} - ${row.AccountManager}`;
            if (!countMap.has(key)) {
              countMap.set(key, 0);
            }
            countMap.set(key, countMap.get(key) + 1);
          });
  
          // Prepare data for the chart
          countMap.forEach((count, name) => {
            data.labels.push(name);
            data.datasets[0].data.push(count);
            data.datasets[0].backgroundColor.push(getRandomColor()); // Function to generate a random color
          });
        }
      });
    });
  
    function getRandomColor() {
      const letters = '0123456789ABCDEF';
      let color = '#';
      for (let i = 0; i < 6; i++) {
        color += letters[Math.floor(Math.random() * 16)];
      }
      return color;
    }
  </script>
  
  <Pie {data} />
  