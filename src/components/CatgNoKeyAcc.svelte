<script>
    import { onMount, onDestroy } from 'svelte';
    import Papa from 'papaparse';
    import Chart from 'chart.js/auto';

    let chart = null;
    let chartContainer;

    onMount(async () => {
        const response = await fetch('/customers.csv');
        const csvData = await response.text();

        const categoryCounts = {};

        Papa.parse(csvData, {
            header: true,
            complete: (results) => {
                results.data.forEach(row => {
                    const category = row.AccountType;
                    if (category in categoryCounts) {
                        categoryCounts[category]++;
                    } else {
                        categoryCounts[category] = 1;
                    }
                });
                createChart(categoryCounts);
            }
        });
    });

    function createChart(categoryCounts) {
        const ctx = chartContainer.getContext('2d');
        chart = new Chart(ctx, {
            type: 'bar',
            data: {
                labels: Object.keys(categoryCounts),
                datasets: [{
                    label: 'Number of Accounts',
                    data: Object.values(categoryCounts),
                    backgroundColor: Object.keys(categoryCounts).map(() => getRandomColor())
                }]
            },
            options: {
                scales: {
                    y: {
                        beginAtZero: true
                    }
                }
            }
        });
    }

    function getRandomColor() {
        const r = Math.floor(Math.random() * 256);
        const g = Math.floor(Math.random() * 256);
        const b = Math.floor(Math.random() * 256);
        return `rgb(${r}, ${g}, ${b})`;
    }

    onDestroy(() => {
        if (chart) chart.destroy();
    });
</script>

<svelte:head>
    <style>
        canvas {
            max-width: 400px; 
            margin: auto;
        }
    </style>
</svelte:head>

<canvas bind:this={chartContainer}></canvas>


 