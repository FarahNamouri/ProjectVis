<script>
    import { onMount } from 'svelte';
    import Plotly from 'plotly.js-dist-min';
  
    onMount(() => {
      fetch('./order_counts.csv')
        .then(response => response.text())
        .then(text => {
          const lines = text.split('\n');
          const data = lines.slice(1).map(line => line.split(',')); 
  
          const territories = data.map(item => item[0]);
          const orderCounts = data.map(item => parseInt(item[1]));
  
          const plotData = [{
            x: territories,
            y: orderCounts,
            type: 'scatter',
            mode: 'lines+markers',
            fill: 'tozeroy', 
            fillcolor: 'rgba(0, 123, 255, 0.3)', 
            marker: {
              color: 'blue',
              size: 8
            },
            line: {
              shape: 'spline',
              smoothing: 1.3,
              width: 3,
              color: 'rgba(0, 123, 255, 1)'
            }
          }];
  
          Plotly.newPlot('myGraph', plotData, {
            title: 'Territory Performance',
            xaxis: { title: 'Territory', automargin: true },
            yaxis: { title: 'Number of Orders', autorange: true },
            hovermode: 'closest',
            plot_bgcolor: 'rgba(255, 255, 255, 0.95)', 
            margin: { l: 60, r: 30, t: 45, b: 30 },
            showlegend: false,
          }, {
            responsive: true
          });
        })
        .catch(error => {
          console.error('Error loading or processing the CSV file:', error);
          alert('Failed to load data. Please check the console for more details.');
        });
    });
  </script>
  
  <div id="myGraph" style="width:100%; height:400px;"></div>