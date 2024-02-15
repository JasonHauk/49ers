<script>
	import * as d3 from 'd3';

  	export let data = [];
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
	let point_hovered = -1;
	let recorded_mouse_position = {x: 0, y: 0};

	let sources = [
		{text: "biofuel", show: true},
		{text: "coal", show: true},
		{text: "fossil", show: true},
		{text: "gas", show: true},
		{text: "hydro", show: true},
		{text: "low_carbon", show: true},
		{text: "nuclear", show: true},
		{text: "oil", show: true},
		{text: "solar", show: true},
		{text: "wind", show: true},
	];

	const color = {
		'biofuel': '#ff0000',
		'coal': '#ffcc00',
		'gas': '#64e3a1',
		'hydro': '#87FEF8',
		'nuclear': '#B693FE',
		'oil': '#887575',
		'solar': '#FFC0C2',
		'wind': '#0096FF',
		'fossil': '#009933',
		'low_carbon': '#ff00ff',
	};


  	// Filter columns based on the show property in sources
	$: visibleColumns = sources.filter(source => source.show).map(source => source.text);

	// Filter data for 'country'
	$: before = data.filter((d) => d.country === searchQuery);
	$: filteredData = before.filter((d) => d.year >= 1960);
  
	// Check if there are elements in filteredData before extracting columns
	$: columns = filteredData.length > 0 ? Object.keys(filteredData[0]).filter(
	  (key) => key !== 'year' && key !== 'country'
	) : [];
    
	// Adjust scales for 'year' and selected column
    $: x = d3
      .scaleBand()
      .domain(filteredData.map((d) => d.year))
      .range([marginLeft, width - marginRight])
      .padding(0.1);
 
    $: y = d3
      .scaleLinear()
      .domain([0, d3.max(filteredData, (d) => d3.max(columns, column => d[column]))])
      .nice()
      .range([height - marginBottom, marginTop]);
 
    let highlightedPoint = null;
 
    // Adjust axis creation for 'year' and selected column
    $: d3.select(gx).call(d3.axisBottom(x).tickValues(x.domain().filter(year => year % 5 === 0)));
    $: d3.select(gy)
      .call(d3.axisLeft(y).ticks(null, 's'))
      .call((g) =>
        g.selectAll('.tick line')
          .clone()
          .attr('x2', width - marginRight - marginLeft)
          .attr('stroke-opacity', (d) => (d === 0 ? 1 : 0.1))
      );


	// Adjust rendering of lines connecting the dots
	$: d3.select(svg)
        .selectAll('.line-group')
        .data(columns)
        .join('g')
        .attr('class', 'line-group')
        .each(function (column) {
            const line = d3.line()
                .x(d => x(d.year))
                .y(d => isNaN(d[column]) ? null : y(d[column]));

			d3.select(this)
				.append('path')
				.datum(filteredData)
				.attr('class', 'line')
				.attr('d', line)
				.attr('stroke', getColorForColumn(column))
				.attr('stroke-width', 1)
				.attr('fill', 'none')
				.attr('stroke-opacity', 0.7);
        });

	// Adjust rendering of points for 'year' and all columns
	$: d3.select(svg)
        .selectAll('g.column-group')
        .data(filteredData)
        .join('g')
        .attr('class', 'column-group')
        .each(function (d) {
            // Filter out NaN values before binding data to circles
            const filteredColumns = visibleColumns.filter(column => !isNaN(d[column]));

            d3.select(this)
                .selectAll('circle')
                .data(filteredColumns)
                .join('circle')
                .attr('cx', (column) => x(d.year))
                .attr('cy', (column) => y(d[column]))
                .attr('r', 2.5)
                .attr('fill', (column) => getColorForColumn(column))
                //   added this
                .on('mouseover', (event, column) => {
                    point_hovered = { year: d.year, value: y(d[column]), column: [column] };
                    highlightedPoint = { year: d.year, [column]: d[column] };
                    tooltipPt = { year: d.year, [column]: d[column] };
                    recorded_mouse_position = {
                        x: event.pageX,
                        y: event.pageY
                    };
                })
                .on('mouseout', () => {
                    point_hovered = -1;
                    highlightedPoint = null;
                    tooltipPt = null;
                });
        });
  
	// Function to get color based on the selected column
	function getColorForColumn(column) {
		return color[column];
	}

	// Add a function to calculate the left position dynamically
    function getLeftPosition() {
        const tooltipWidth = 60; // Adjust the width based on your tooltip's content
        const screenWidth = window.innerWidth;
        const cursorX = recorded_mouse_position.x + 10 || 0;
        
        // Check if the cursor is on the right half of the screen
        if (cursorX > screenWidth / 2) {
            return cursorX - tooltipWidth - 10; // Adjust 20 as needed for spacing
        } else {
            return cursorX; // Default position to the right
        }
    }

	
  // ... (previous code)

  // Function to find the closest point to the cursor
  function findClosestPoint(event, data) {
    const filteredColumns = visibleColumns.filter(column => !isNaN(data[column]));

    const distances = filteredColumns.map(column => ({
      column,
      distance: Math.abs(y(data[column]) - event.pageY)
    }));

    const closestPoint = distances.reduce((prev, curr) =>
      prev.distance < curr.distance ? prev : curr
    );

    return {
      year: data.year,
      value: y(data[closestPoint.column]),
      column: [closestPoint.column]
    };
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
		  Electricity Per Captia
		</text>
	  </g>
	</svg>
	
	<!-- tooltip -->
	<div
		class={point_hovered === -1 ? "tooltip-hidden" : "tooltip-visible"}
		style={`left: ${getLeftPosition()}px; 
				top: ${recorded_mouse_position.y + 10 || 0}px; 
				background-color: ${getColorForColumn(point_hovered.column) || 'rgba(233, 179, 215, 0.533)'};`}
	>
		{#if point_hovered !== -1}
			<div style="display: flex; 
						flex-direction: column; 
						align-items: flex-start; 
						min-width: fit-content;">
				<!-- <div>Source: {point_hovered.column}</div> -->
				<div>Year: {point_hovered.year}</div>
				<div>Value: {point_hovered.value.toFixed(2)}</div>
			</div>
		{/if}
	</div>
  </div>
  
  <div class="source" style="display: flex; flex-wrap: wrap;">
	{#each sources as source}
		<div class="source-item" style="background-color: {source.show ? color[source.text] : 'white'} ;">
			<input
            	bind:checked={source.show}
            	id={source.text}
            	type="checkbox"
            	class="checkbox-round"
        	/>
			<label style="color: black;">{source.text}</label>
		</div>
	{/each}
  </div>

  <style>
	.source {
		display: flex;
		justify-content: center;
		flex-wrap: wrap;
	}

	.source-item {
		display: inline-block;
        padding: 0 5px;
		height: 50px;
        border-radius: 10px;
    	margin: 0 5px; /* Adjust margin between items if needed */
  	}

	.tooltip-hidden {
		visibility: hidden;
		font-family: "Nunito", sans-serif;
		width: 200px;
		position: absolute;
	}

    .tooltip-visible {
        font: 12px sans-serif;
        font-family: "Nunito", sans-serif;
        visibility: visible;
        border-radius: 10px;
        width: 80px;
        color: black;
        position: absolute;
        padding: 5px;
    }
	
</style>