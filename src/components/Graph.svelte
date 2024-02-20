<script>
	import * as d3 from 'd3';

  	export let data = [];
  	export let searchQuery = '';
  
	const width = 928;
	const height = 600;
	const marginTop = 30;
	const marginRight = 30;
	const marginBottom = 60;
	const marginLeft = 70;
  
	let svg;
	let gx;
	let gy;
	let point_hovered = -1;
	let max_year = -1;
	let recorded_mouse_position = {x: 0, y: 0};

	let sources = [
		{text: "biofuel", show: true},
		{text: "coal", show: true},
		{text: "gas", show: true},
		{text: "hydro", show: true},
		{text: "nuclear", show: true},
		{text: "oil", show: true},
		{text: "solar", show: true},
		{text: "wind", show: true},
		{text: "fossil", show: false},
		{text: "low_carbon", show: false},
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
      .domain([0, d3.max(filteredData, (d) => d3.max(visibleColumns, column => d[column]))])
      .nice()
      .range([height - marginBottom, marginTop]);
 
 
    // Adjust axis creation for 'year' and selected column
    $: d3.select(gx).call(d3.axisBottom(x).tickValues(x.domain().filter(year => year % 5 === 0)));
    $: { d3.select(gy).selectAll("*").remove()
		d3.select(gy)
      .call(d3.axisLeft(y).ticks(null, 's'))
      .call((g) =>
        g.selectAll('.tick line')
          .clone()
          .attr('x2', width - marginRight - marginLeft)
          .attr('stroke-opacity', (d) => (d === 0 ? 1 : 0.1))
      );
	}

	// Adjust rendering of lines connecting the dots
	$: {
        let visibleColumnsFiltered = columns.filter(column => sources.find(source => source.text === column && source.show));
		d3.select(svg)
        .selectAll('.line-group')
        .data(visibleColumnsFiltered, d => d)
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
	}

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
                .attr('cx', x(d.year))
                .attr('cy', (column) => y(d[column]))
                .attr('r', 2.5)
                .attr('fill', (column) => getColorForColumn(column))

        })
		
	// area of tooltip
	$: d3.select(svg)
        .selectAll('g.column-group')
        .data(filteredData)
        .join('g')
        .attr('class', 'column-group')
        .each(function (d) {
            // Filter out NaN values before binding data to circles
            const filteredColumns = visibleColumns.filter(column => !isNaN(d[column]));

			d3.select(this)
				.selectAll('circle.new')
				.data(filteredColumns)
				.join('circle')
				.attr('class', 'new')
				.attr('cx', x(d.year))
				.attr('cy', (column) => y(d[column]))
				.attr('r', 6.5) // Adjust the radius as needed
				.attr('fill', 'red') // Set the color for the new set
				.attr('fill-opacity', 0.0) // Set the opacity for the new set
				.attr('clip-path', 'url(#clip-path)')
				.on('pointerenter pointermove', (event, column) => {
					point_hovered = { year: d.year, value: d[column], column: [column] };
					recorded_mouse_position = {
						x: event.pageX,
						y: event.pageY
					};
				})
				.on('pointerleave', () => {
					point_hovered = -1;
				});
        })
	
	// max source tooltip
	$: d3.select(svg)
        .selectAll('g.column-group')
        .data(filteredData)
        .join('g')
        .attr('class', 'column-group')
        .each(function (d) {
			d3.select(this)
				.on('pointerenter pointermove', (event, column) => {
					max_year = { ttmax: d.tt };
				})
        })
  
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
			// console.log(cursorX)
            return cursorX; // Default position to the right
        }
    }
	$: {
		console.log(filteredData[62].tt)
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
	  <!-- x-axis label -->
		<text
			x={(width - marginLeft - marginRight) / 2 + marginRight + 55}
			y={height - marginBottom / 2}
			dy="1.5em"
			fill="#000"
			font-weight="bold"
			font-size="16px"
			text-anchor="middle"
		>
			Year
		</text>
	  <!-- y-axis -->
	  <g bind:this={gy} transform="translate({marginLeft},0)"/>
	  <g transform="translate({marginLeft},0)">
    	<!-- y-axis label -->
    	<text
      		x={-(height - marginRight - marginLeft) / 2 - marginRight}
      		y={-marginLeft + marginRight - 10}
      		dy="0em"
      		fill="#000"
      		font-weight="bold"
			font-size="16px"
      		text-anchor="middle"
      		transform="rotate(-90)"
    	>
      		Electricity Per Capita in Kilowatt Hours (kWh)
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
				<div>Year: {point_hovered.year}</div>
				<div>Value: {point_hovered.value.toFixed(2)} kWh</div>
			</div>
		{/if}
	</div>
  </div>

  <div
		class={"tooltip-visible"}
		style={`left: 1200px; 
				top: 200px; 
				};`}
	>
		{#if max_year !== -1}
			<div style="display: flex; 
						flex-direction: column; 
						align-items: flex-start; 
						min-width: fit-content;
						font-size: 16px; /* Adjust the font size as needed */">
				<div>{max_year.ttmax}</div>
			</div>
		{:else}
			<div style="display: flex; 
					flex-direction: column; 
					align-items: flex-start; 
					min-width: fit-content;
					font-size: 16px; /* Adjust the font size as needed */">
			<div>{filteredData[62].tt}</div>
			</div>
		{/if}
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
        padding: 0px 5px;
		height: 50px;
        border-radius: 10px;
    	margin: 5px 5px; /* Adjust margin between items if needed */
  	}

	.tooltip-hidden {
		visibility: hidden;
		font-family: "Nunito", sans-serif;
		width: 200px;
		position: absolute;
	}

	.tt-hidden {
        visibility: hidden;
        opacity: 0; /* Start with opacity set to 0 */
        transition: opacity 2s ease-out; /* Add a fade-out transition over 0.5 seconds */
        font-family: "Nunito", sans-serif;
        width: 200px;
        position: absolute;
    }
    .tooltip-visible {
        font: 12px sans-serif;
        font-family: "Nunito", sans-serif;
        visibility: visible;
        border-radius: 10px;
        width: 120px;
        color: black;
        position: absolute;
        padding: 7px;
    }
	
</style>