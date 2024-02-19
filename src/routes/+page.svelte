<h1>Welcome to SvelteKit!!</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

<script>

    import * as d3 from 'd3';
    
    import { onMount } from 'svelte';

    let energyData = [];
    let emissionData = []

    // onMount(async () => {

    //    const res = await fetch('raw.githubusercontent.com_owid_energy-data_master_owid-energy-data.csv'); 

    //    const csv = await res.text();

    //    energyData = d3.csvParse(csv, d3.autoType)

    //    console.log(energyData);

    // });
    onMount(async () => {

        const res = await fetch('Carbon Emission.csv'); 

        const csv = await res.text();

        emissionData = d3.csvParse(csv, d3.autoType)

        $: console.log(emissionData);
        createBubbleChart();

    });

    function createBubbleChart() {
        // Define SVG dimensions and margins
        const width = 900;
        const height = 800;
        const margin = { top: 50, right: 50, bottom: 70, left: 70 };

        // Append SVG to the HTML document
        const svg = d3.select("svg")
                      .attr("width", width)
                      .attr("height", height);

        // Find maximum values for x and y axes
        const maxX = d3.max(emissionData, d => d["Vehicle Monthly Distance Km"]);
        const maxY = d3.max(emissionData, d => d.CarbonEmission);

        // Create linear scales for x and y axes
        const xScale = d3.scaleLinear()
                         .domain([0, maxX + 1000])
                         .range([margin.left, width - margin.right]);

        const yScale = d3.scaleLinear()
                         .domain([0, maxY + 1000])
                         .range([height - margin.bottom, margin.top]);

        const radiusScale = d3.scaleSqrt()
                              .domain([0, d3.max(emissionData, d => d["Waste Bag Weekly Count"])])
                              .range([1, 10]); // Adjust as needed

        // Add circles for each data point
        svg.selectAll("circle")
           .data(emissionData)
           .enter()
           .append("circle")
           .attr("cx", d => xScale(d["Vehicle Monthly Distance Km"]))
           .attr("cy", d => yScale(d.CarbonEmission))
           .attr("r", d => radiusScale(d["Waste Bag Weekly Count"]))
           .attr("fill", "steelblue")
           .attr("opacity", 0.7);

        // Add labels for x and y axes
        svg.append("g")
           .attr("transform", `translate(0, ${height - margin.bottom})`)
           .call(d3.axisBottom(xScale));

        svg.append("g")
           .attr("transform", `translate(${margin.left}, 0)`)
           .call(d3.axisLeft(yScale));

        // Add X axis label
        svg.append("text")
           .attr("text-anchor", "end")
           .attr("x", width - margin.right)
           .attr("y", height - 10)
           .text("Vehicle Monthly Distance Km");

        // Add Y axis label
        svg.append("text")
           .attr("text-anchor", "end")
           .attr("transform", "rotate(-90)")
           .attr("y", 10)
           .attr("x", -margin.top)
           .text("Carbon Emission");

        // Add tooltip
        const tooltip = d3.select("body").append("div")
                          .attr("class", "tooltip")
                          .style("opacity", 0);

        svg.selectAll("circle")
           .on("mouseover", function(event, d) {
               tooltip.transition()
                      .duration(200)
                      .style("opacity", .9);
               tooltip.html(`Carbon Emission: ${d.CarbonEmission}<br>Internet Daily Hour: ${d["How Long Internet Daily Hour"]}<br>Clothes Monthly: ${d["How Many New Clothes Monthly"]}`)
                      .style("left", (event.pageX) + "px")
                      .style("top", (event.pageY - 28) + "px");
           })
           .on("mouseout", function(d) {
               tooltip.transition()
                      .duration(500)
                      .style("opacity", 0);
           });
    }
</script>

<svg style="display:block;margin:auto;"></svg>

<style>
  svg {
    border: 1px solid #ccc;
  }
</style>
