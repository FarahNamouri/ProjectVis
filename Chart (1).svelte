<script>
  import { onMount } from "svelte";
  import { csv } from "d3-fetch";
  import { scaleBand, scaleLinear } from "d3-scale";
  import { select, selectAll } from "d3-selection";
  import { axisTop, axisLeft } from "d3-axis";
  import { extent, max, range } from "d3-array";
  import { line } from "d3-shape";

  import App from "../App.svelte";

  let data = [];
  let tooltipData;

  $: tooltipData;

  function countItem(array, item) {
    let count = 0;
    for (let i = 0; i < array.length; i++) {
      if (array[i] === item) {
        count++;
      }
    }
    return count;
  }
  function flattenArray(arr) {
    return arr.reduce((acc, val) => {
      return acc.concat(val);
    }, []);
  }

  onMount(async () => {
    let fetchData = await csv("../../Book1.xlsx.csv", (data) => ({
      customer: data["Account Code"],
      territory: data["Territory"],
    }));
    data = [...fetchData];
  });

  $: allTerritories = data.map((d) => d.territory);
  $: territories = [...new Set(allTerritories)];

  $: counts = territories.map((item) => {
    let obj = {};
    obj.territory = item;
    obj.count = countItem(allTerritories, item);
    return obj;
  });

  const margin = { top: 20, right: 40, bottom: 80, left: 160 },
    width = 1200 - margin.left - margin.right,
    height = 3600 - margin.top - margin.bottom;

  //xscale
  $: x = scaleBand().domain(range(1,25)).range([0, width]).padding(0.1);

  //yscale
  $: y = scaleBand().domain(territories).range([0, height]).padding(0.1);

  $: {
    const svg = select("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    const yAxis = svg
      .append("g")
      .attr("class", "y-axis")
      .call(axisLeft(y))
      .style("font-size", "0.875em");

    const xAxis = svg
      .append("g")
      .attr("class", "x-axis")
      .call(axisTop(x))
      .style("font-size", "0.875em");

    //removing ticks and axis lines
    selectAll(".y-axis  path").style("visibility", "hidden");
    selectAll(".y-axis line").style("visibility", "hidden");

    selectAll(".x-axis  path").style("visibility", "hidden");
    selectAll(".x-axis line").style("visibility", "hidden");

    // line scales
    const xLineScale = scaleLinear().domain(range(25)).range([0, width]);
    const yLineScale = scaleLinear().domain(range(212)).range([height, 0]);

    xAxis.selectAll("path").style("stroke", "white");
    xAxis.selectAll("text").style("fill", "white");

    yAxis.selectAll("path").style("stroke", "white");
    xAxis.selectAll("text").style("fill", "white");
    yAxis.selectAll("text").style("fill", "white");

    //title 
    svg
      .append("text")
      .attr("x", width / 2)
      .attr("y", -100)
      .text("Customer Territorial Distribution Chart")
      .attr("text-anchor", "middle")
      .style("fill", "white")
      .style("font-size", "1.825em");


    svg
      .append("text")
      .attr("x", width / 2)
      .attr("y", -40)
      .text("Number of Customers Per Territories")
      .attr("text-anchor", "middle")
      .style("fill", "white")
      .style("font-size", "1.25em");

    svg
      .append("text")
      .attr("x", -height / 2)
      .attr("y", -135)
      .attr("transform", "rotate(-90)")
      .text("Territories")
      .attr("text-anchor", "middle")
      .style("fill", "white")
      .style("font-size", "1.25em");

    //color scale
    const colorScale = scaleLinear()
      .domain(extent(counts, (d) => d.count))
      .range(["#DCF0FB", "#106A9E"]);

    //preparin data for forground rects
    let fillerData = [];

    for (let j = 1; j < 25; j++) {
      let items = [];
      for (let i = 0; i < 212; i++) {
        let obj = {};
        obj.count = j;
        obj.territory = territories[i];
        items.push(obj);
      }
      fillerData.push(items);
    }

    let flatData = flattenArray(fillerData);

    const colorScaleForeground = scaleLinear()
      .domain(extent(flatData, (d) => d.count))
      .range(["#AFD198", "#114232"]);

    function mouseleave(e) {
      tooltip.style("opacity", 0);
      select(this).style("stroke", "none").style("opacity", 0.8);
    }

    const handleMouseMove = (e, d) => {
      tooltip.style("opacity", 1);

      tooltip
        .style("left", e.pageX - 40 + "px")
        .style("top", e.pageY - 40 + "px");
    };

    function mouseenter(e, d) {
      tooltip.style("opacity", 1);
      tooltipData = d;

      select(this)
        .style("stroke", "#C97B0D")
        .style("stroke-width", 1.5)
        .style("opacity", 1);

      tooltip
        .style("left", e.pageX - 40 + "px")
        .style("top", e.pageY - 40 + "px");
    }

    svg
      .selectAll()
      .data(flatData)
      .enter()
      .append("rect")
      .attr("x", (d) => x(d.count))
      .attr("y", (d) => y(d.territory))
      .attr("width", x.bandwidth())
      .attr("height", y.bandwidth())
      .attr("fill", (d) => colorScaleForeground(d.count));

    svg
      .selectAll()
      .data(counts)
      .enter()
      .append("rect")
      .attr("x", (d) => x(d.count))
      .attr("y", (d) => y(d.territory))
      .attr("width", x.bandwidth())
      .attr("height", y.bandwidth())
      .attr("fill", (d) => colorScale(d.count))
      .on("mouseenter", mouseenter)
      .on("mousemove", handleMouseMove)
      .on("mouseleave", mouseleave);

    svg
      .selectAll()
      .data(counts)
      .enter()
      .append("text")
      .attr("x", (d) => x(d.count) + 6)
      .attr("y", (d) => y(d.territory) + 11)
      .attr("fill", "white")
      .style("font-size", "0.575em")
      .text((d, i) => `${i+1},${d.count}`)
      .style("fill", "#6420AA")
      .on("mouseenter", mouseenter)
      .on("mousemove", handleMouseMove)
      .on("mouseleave", mouseleave);

      svg
      .selectAll()
      .data(counts)
      .enter()
      .append("text")
      .attr("x", width+ 10)
      .attr("y", (d,i) => i *16 +10)
      .attr("fill", "white")
      .style("font-size", "0.675em")
      .text((d, i) => `${d?.territory} - [${i+1},${d?.count}]`)

    const tooltip = select("#tooltip")
      .style("opacity", 0)
      //.attr("class", "tooltip")
      .style("background-color", "#795458")
      .style("padding", "5px")
      .style("padding-bottom", "10px")
      .style("position", "absolute")
      .style("font-size", "0.75em");
  }
</script>

<main>
  <div class="relative">
    <div id="tooltip" class="tooltip">
      <div class="inline-block text-lg">Count: {tooltipData?.count}</div>
      <div class="inline-block">
        Territory: {tooltipData?.territory}
      </div>
    </div>
    <svg  id="svg" viewBox={`0 0 ${width + 300} ${height}`}></svg>
  </div>
</main>

<style>
  svg {
    /* background-color: #50727B; */
    /* background-color: #1E0342; */
  }
  .relative {
    position: relative;
  }
  #tooltip {
    font-size: 0.575em;
    opacity: 0;
  }
</style>
