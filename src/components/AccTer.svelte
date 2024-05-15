<script>
    import { onMount } from 'svelte';
    import Papa from 'papaparse';

    let customersPerTerritory = [];

    onMount(async () => {
        const response = await fetch('/customers.csv');
        const csvData = await response.text();

        Papa.parse(csvData, {
            header: true,
            complete: (results) => {
                const territoryCounts = results.data.reduce((acc, row) => {
                    const territory = row.Territory; 
                    if (territory) {
                        if (!acc[territory]) {
                            acc[territory] = 0;
                        }
                        acc[territory]++;
                    }
                    return acc;
                }, {});

                customersPerTerritory = Object.entries(territoryCounts)
                    .sort((a, b) => a[0].localeCompare(b[0])); 
            }
        });
    });
</script>

<table>
    <thead>
        <tr>
            <th>Territory</th>
            <th>Number of Customers</th>
        </tr>
    </thead>
    <tbody>
        {#each customersPerTerritory as [territory, count]}
        <tr>
            <td>{territory}</td>
            <td>{count}</td>
        </tr>
        {/each}
    </tbody>
</table>

<style>
    table {
        width: 100%;
        border-collapse: collapse;
    }
    th, td {
        border: 1px solid #ccc;
        padding: 8px;
        text-align: left;
    }
    th {
        background-color: #eee;
    }
</style>



