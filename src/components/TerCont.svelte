<script>
  import { onMount } from 'svelte';

  let data = [];
  let maxIncome = 0; 
  let totalIncome = 0; // income from all territories

  onMount(async () => {
    const response = await fetch('./TerIncomeCompressed.csv');
    if (!response.ok) {
      console.error('Failed to fetch data:', response.status);
      return;
    }

    const csvText = await response.text();
    const rows = csvText.split('\n').slice(1); //skipping header
    let allData = rows.map(row => {
      const [territory, cartPriceInCP] = row.trim().split(',');
      const cleanedCartPrice = parseFloat(cartPriceInCP);
      if (isNaN(cleanedCartPrice)) {
        console.error('Failed to parse:', cartPriceInCP); 
        return { territory, cartPriceInCP: 0 };
      }
      return { territory, cartPriceInCP: cleanedCartPrice };
    });

    totalIncome = allData.reduce((acc, { cartPriceInCP }) => acc + cartPriceInCP, 0);
    data = allData.slice(0, 10); // first 10 territories
    maxIncome = Math.max(...data.map(({ cartPriceInCP }) => cartPriceInCP)); // max income 
  });
</script>

<svg width="1000" height="550">
  <text x="10" y="20" font-size="16" fill="blue">Total Income: {totalIncome}</text>
  
  {#each data as row, index (row.territory)}
    <g transform={`translate(0, ${50 + index * 50})`}>
      <rect x="200" y="10" width={row.cartPriceInCP / maxIncome * 750} height="30" fill="blue" />
      <text x="10" y="30" fill="black">{row.territory}</text>
      <text x={200 + row.cartPriceInCP / maxIncome * 750 + 10} y="30" fill="black">{row.cartPriceInCP}</text>
    </g>
  {/each}
</svg>

<style>
  rect {
    transition: fill 0.3s ease;
    fill: #a2d2ff;
  }
  rect:hover {
    fill: #00b4d8; 
  }
  text {
    font-family: Arial, sans-serif;
  }
</style> 

