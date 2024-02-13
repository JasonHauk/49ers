<script>
	import * as d3 from 'd3';

  	export let data = [];
  	export let selectedColumn = 'biofuel_elec_per_capita';
  	export let searchQuery = '';
  
	const width = 928;
	const height = 600;
	const marginTop = 20;
	const marginRight = 30;
	const marginBottom = 30;
	const marginLeft = 40;
  
	let svg;
	let gx;
	let gy;
	let tooltipPt = null;
  
	// Filter data for 'United States'
	$: before = data.filter((d) => d.country === searchQuery);
	$: filteredData = before.filter((d) => d.year >= 1960);
  
	// Check if there are elements in filteredData before extracting columns
	$: columns = filteredData.length > 0 ? Object.keys(filteredData[0]).filter(
	  (key) => key !== 'year' && key !== 'country'
	) : [];
  
	// Define a color scale for columns (check if columns array is not empty)
	$: colorScale = columns.length > 0 ? d3.scaleOrdinal().domain(columns).range(d3.schemeCategory10) : null;
  
	// Adjust scales for 'year' and selected column
	$: x = d3
	  .scaleLinear()
	  .domain(d3.extent(filteredData, (d) => d.year))
	  .range([marginLeft, width - marginRight]);
  
	$: y = d3
	  .scaleLinear()
	  .domain([0, d3.max(filteredData, (d) => d[selectedColumn])])
	  .nice()
	  .range([height - marginBottom, marginTop]);
  
	let highlightedPoint = null;
  
	// Adjust axis creation for 'year' and selected column
	$: d3.select(gx).call(d3.axisBottom(x).ticks(width / 80));
	$: d3.select(gy)
	  .call(d3.axisLeft(y).ticks(null, '+'))
	  .call((g) =>
		g
		  .selectAll('.tick line')
		  .clone()
		  .attr('x2', width - marginRight - marginLeft)
		  .attr('stroke-opacity', (d) => (d === 0 ? 1 : 0.1)),
	  );
  
	// Adjust rendering of points for 'year' and all columns
	$: d3.select(svg)
  .selectAll('g.column-group')
  .data(filteredData)
  .join('g')
  .attr('class', 'column-group')
  .each(function (d) {
    // Filter out NaN values before binding data to circles
    const filteredColumns = columns.filter(column => !isNaN(d[column]));

    d3.select(this)
      .selectAll('circle')
      .data(filteredColumns)
      .join('circle')
      .attr('cx', (column) => x(d.year))
      .attr('cy', (column) => y(d[column]))
      .attr('r', 2.5)
      .attr('fill', (column) => getColorForColumn(column))
      .on('mouseover', (event, column) => {
  highlightedPoint = { year: d.year, [column]: d[column] };
})
      .on('mouseout', () => {
        highlightedPoint = null;
      });
  });
  
	// Function to get color based on the selected column
	function getColorForColumn(column) {
	  return colorScale ? colorScale(column) : 'steelblue';
	}
  </script>
  
  <div class="population-plot">
	<svg
	  bind:this={svg}
	  {width}
	  {height}
	  viewBox="0 0 {width} {height}"
	  style="max-width: 100%; height: auto;"
	>
	  <!-- x-axis -->
	  <g bind:this={gx} transform="translate(0,{height - marginBottom})" />
	  <!-- y-axis -->
	  <g bind:this={gy} transform="translate({marginLeft},0)">
		<text
		  x="5"
		  y={marginTop}
		  dy="0.32em"
		  fill="#000"
		  font-weight="bold"
		  text-anchor="start"
		>
		  Population
		</text>
	  </g>
  
	  <!-- points -->
	  <g stroke="#000" stroke-opacity="0.2">
		{#each filteredData as d, i}
		  <circle
			key={i}
			cx={x(d.year)}
			cy={y(d[selectedColumn])}
			fill={(highlightedPoint === d ? 'red' : 'steelblue')}
			r="2.5"
		  />
		{/each}
	  </g>
  
	  <!-- tooltip -->
	  {#if tooltipPt}
		<g transform="translate({x(tooltipPt.year)},{y(tooltipPt[selectedColumn])})">
		  <text font-weight="bold">{tooltipPt[selectedColumn]}</text>
		</g>
	  {/if}
	</svg>
  
	<!-- legend -->
	<div class="legend">
	  {#if colorScale}
		{#each columns as column}
		  <div class="legend-item">
			<span class="legend-color" style="background-color: {colorScale(column)};"></span>{column}
		  </div>
		{/each}
	  {/if}
	</div>
  </div>
  
  <style>
	.legend {
	  text-align: center;
	  margin-top: 10px;
	}
  
	.legend-item {
	  display: inline-block;
	  margin-right: 10px;
	}
  
	.legend-color {
	  display: inline-block;
	  width: 16px;
	  height: 16px;
	  margin-right: 4px;
	}
</style>
