
<head>
<style>
  body {
    background-color: #cccccc;
     /* Center aligning the text */
     text-align: right;
  }
  h1 {
    color: black; /* Changing text color */
    text-align: center;
  }
  p {
    color: black; /* Changing text color */
    font-size: 20px;
    text-align: center;
  }
</style>
</head>
<body>

<h1>Shower Power: Scrubbing the Earth Clean or Washing Away Our Future?</h1>
<p>Exploring the Interplay Between Shower Frequency, Waste Generation, and Carbon Emissions</p>

</body>

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

        // $: console.log(emissionData);
        createBubbleChart();

    });

    function createBubbleChart() {
        // Define SVG dimensions and margins
        const width = 1000;
        const height = 800;
        const margin = { top: 50, right: 50, bottom: 70, left: 70 };

        // Append SVG to the HTML document
        const svg = d3.select("svg")
                      .attr("width", width)
                      .attr("height", height)
                      ;

        // Find maximum values for x and y axes
        const maxX = d3.max(emissionData, d => d["Vehicle Monthly Distance Km"]);
        const maxY = d3.max(emissionData, d => d.CarbonEmission);

        // Create linear scales for x and y axes
        const xScale = d3.scaleLinear()
                         .domain([-500, maxX + 1000])
                         .range([margin.left, width - margin.right]);

        const yScale = d3.scaleLinear()
                         .domain([0, maxY + 1000])
                         .range([height - margin.bottom, margin.top]);

        const radiusScale = d3.scaleSqrt()
                              .domain([0, d3.max(emissionData, d => d["How Many New Clothes Monthly"])])
                              .range([2, 20]); // Adjust as needed

        const categories = Array.from(new Set(emissionData.map(d => d["How Often Shower"])));
        //console.log(categories);

        const colorScale = d3.scaleOrdinal()
                         .domain(categories)
                         .range(["#2F9599", "#E84A5F", "#F7DB4F", "#F26B38", "#355C7D", "#2F9599"]);

        // Add circles for each data point
        const circles = svg.selectAll("circle")
           .data(emissionData)
           .enter()
           .append("circle")
           .attr("cx", d => xScale(d["Vehicle Monthly Distance Km"]))
           .attr("cy", d => yScale(d.CarbonEmission))
           .attr("r", d => radiusScale(d["How Many New Clothes Monthly"]))
           .attr("fill", d => colorScale(d["How Often Shower"]))
           .attr("opacity", 0.7);
        

        // Add labels for x and y axes
        svg.append("g")
           .attr("transform", `translate(0, ${height - margin.bottom})`)
           .call(d3.axisBottom(xScale).tickValues(d3.range(0, maxX + 600, 500)));


        svg.append("g")
           .attr("transform", `translate(${margin.left}, 0)`)
           .call(d3.axisLeft(yScale));

        // Add X axis label
        svg.append("text")
           .attr("text-anchor", "middle")
           .attr("x", width/2)
           .attr("y", height - 10)
           .text("Vehicular Monthly Distance (Km)");

        // Add Y axis label
        svg.append("text")
           .attr("text-anchor", "middle")
           .attr("transform", "rotate(-90)")
           .attr("y", margin.left / 3) // Position at the center of the SVG vertically
           .attr("x", -height / 2) // Position at the center of the SVG horizontally (negative because of rotation)
           .text("Carbon Emission");

        const legend = svg.selectAll(".legend")
            .data(categories)
            .enter().append("g")
            .attr("class", "legend")
            .attr("transform", function(d, i) { return "translate(0," + i * 20 + ")"; })
            .on("mouseover", function(event, d) { updateVisualization(event, d); })
            .on("click", function(event, d) { updateVisualization(event, d); });

        // Add colored rectangles to legend
        legend.append("rect")
            .attr("x", width - margin.right - 18)
            .attr("width", 18)
            .attr("height", 18)
            .style("fill", colorScale);

        // Add text to legend
        legend.append("text")
            .attr("x", width - margin.right - 24)
            .attr("y", 9)
            .attr("dy", ".35em")
            .style("text-anchor", "end")
            .text(function(d) {
            if (d === 'Less frequently') {
                return capitalizeFirstLetter("Less Than Once a Day");
            } else if (d === 'More frequently') {
                return capitalizeFirstLetter("More Than Twice a Day");
            } else {
                return capitalizeFirstLetter(d === null ? "Unknown" : d);
            } }
            );

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
                      .style("right", (event.pageX) + "px")
                      .style("top", (event.pageY - 28) + "px");
           })
           .on("mouseout", function(d) {
               tooltip.transition()
                      .duration(500)
                      .style("opacity", 0);
           });

        function updateVisualization(event, category) {
            const selectedCategory = category === "Unknown" ? null : category;
            circles.attr("display", function(d) {
                return d["How Often Shower"] === selectedCategory ? "block" : "none";
            });
        }
        
        function resetVisualization() {
            d3.selectAll("circle").attr("display", "block");
        }

        function capitalizeFirstLetter(string) {
            return string.charAt(0).toUpperCase() + string.slice(1);
        }
    }

    
</script>

<svg style="display:block;margin:auto;"></svg>

