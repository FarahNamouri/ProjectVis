<script>
    import { onMount, onDestroy } from 'svelte';
    import Papa from 'papaparse';
    import Chart from 'chart.js/auto';

    let chart = null;
    let chartContainer;
    let chartData = {
        labels: [],
        datasets: [{
            data: [],
            backgroundColor: [],
        }]
    };

    onMount(async () => {
        const response = await fetch('/customers.csv');
        const csvData = await response.text();

        Papa.parse(csvData, {
            header: true,
            complete: (results) => {
                const managerCounts = new Map();

                results.data.forEach((row) => {
                    const manager = row.AccountManager;
                    const accountName = row.AccountName;
                    if (!managerCounts.has(manager)) {
                        managerCounts.set(manager, new Set());
                    }
                    managerCounts.get(manager).add(accountName);
                });

                chartData.labels = [];
                chartData.datasets[0].data = [];
                chartData.datasets[0].backgroundColor = [];
                
                const sortedManagers = Array.from(managerCounts)
                    .map(([manager, accounts]) => ({ manager, count: accounts.size }))
                    .sort((a, b) => b.count - a.count)
                    .slice(0, 10);

                sortedManagers.forEach(({ manager, count }) => {
                    chartData.labels.push(manager);
                    chartData.datasets[0].data.push(count);
                    chartData.datasets[0].backgroundColor.push(getRandomColor());
                });

                if (chartContainer) {
                    chart = new Chart(chartContainer, {
                        type: 'pie',
                        data: chartData,
                        options: {
                            responsive: true,
                            plugins: {
                                legend: {
                                    display: false 
                                },
                                tooltip: {
                                    enabled: true
                                }
                            }
                        }
                    });
                }
            }
        });
    });

    function getRandomColor() {
        const r = Math.floor(Math.random() * 256);
        const g = Math.floor(Math.random() * 256);
        const b = Math.floor(Math.random() * 256);
        return `rgb(${r}, ${g}, ${b})`;
    }

    onDestroy(() => {
        if (chart) {
            chart.destroy();
        }
    });
</script>

<div class="chart-area">
    <h1>Account distribution across managers: who's the top manager?</h1>
    <canvas bind:this={chartContainer}></canvas>
    <div class="legend-box">
        {#each chartData.labels as label, index (label)}
            <div class="legend-item">
                <span class="color-indicator" style="background-color: {chartData.datasets[0].backgroundColor[index]}"></span>
                {label}: {chartData.datasets[0].data[index]}
            </div>
        {/each}
    </div>
</div>

<style>
    .chart-area {
        display: flex;
        flex-direction: column; 
        align-items: center; 
        justify-content: center; 
        padding: 20px;
    }
    .legend-box {
        padding: 10px;
        border: 1px solid #ccc;
        background-color: white;
        box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        margin-top: 20px; 
    }
    .legend-item {
        display: flex;
        align-items: center;
        margin-bottom: 5px;
    }
    .color-indicator {
        width: 20px;
        height: 20px;
        display: inline-block;
        margin-right: 10px;
    }
    canvas {
        max-width: 200px;
        max-height: 200px;
    }
</style>
