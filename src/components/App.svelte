<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Graph from '../components/Graph.svelte';
  import { Runtime, Inspector } from "@observablehq/runtime";
  import define from "@d3/versor-dragging";

  let data = [];
  let selectedColumn = 'biofuel_elec_per_capita';
  let searchQuery = '';
  let isLoading = true;
  let loadingError = null;
  let chartContainer;

  onMount(async () => {
    try {
      const res = await fetch(
        'https://nyc3.digitaloceanspaces.com/owid-public/data/energy/owid-energy-data.csv',
      );
      const csv = await res.text();
      data = d3.csvParse(csv, (d) => {
        return {
          year: +d.year,
          country: d.country,
          biofuel_elec_per_capita: +d.biofuel_elec_per_capita,
          coal_elec_per_capita: +d.coal_elec_per_capita,
          fossil_elec_per_capita: +d.fossil_elec_per_capita,
          gas_elec_per_capita: +d.gas_elec_per_capita,
          hydro_elec_per_capita: +d.hydro_elec_per_capita,
          low_carbon_elec_per_capita: +d.low_carbon_elec_per_capita,
          nuclear_elec_per_capita: +d.nuclear_elec_per_capita,
          oil_elec_per_capita: +d.oil_elec_per_capita,
          other_renewables_elec_per_capita: +d.other_renewable_elec_per_capita,
          renewables_elec_per_capita: +d.renewables_elec_per_capita,
          solar_elec_per_capita: +d.solar_elec_per_capita,
          wind_elec_per_capita: +d.wind_elec_per_capita,
        };
      });

      // Initialize the chart container
      chartContainer = document.createElement('div');
      document.body.appendChild(chartContainer);

      const runtime = new Runtime();
      const main = runtime.module(define, (name) => {
        if (name === "chart") {
          return new Inspector(chartContainer);
        }
      });

      isLoading = false;
    } catch (error) {
      loadingError = 'Error loading data. Please try again later.';
      console.error('Data loading error:', error);
    }
  });
</script>

<main>
  <h1>Elec Per Capita</h1>
  <input bind:value={searchQuery} placeholder="Search for a country" />

  {#if isLoading}
    <p>Loading...</p>
  {:else if loadingError}
    <p>{loadingError}</p>
  {:else}
    <Graph {data} {selectedColumn} {searchQuery} />
    {#if chartContainer}
      <div></div> <!-- No need for bind:this here -->
    {/if}
  {/if}
</main>
  
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Nunito:wght@300;400;700&display=swap');
  
    :root {
      --color-bg: #ffffff;
      --color-outline: #c2c2c2;
      --color-shadow: hsl(0, 0%, 0%, 0.1);
      --color-text: #3f4252;
      --color-bg-1: hsla(0, 0%, 0%, 0.2);
      --color-shadow-1: hsl(0, 0%, 96%);
    }
  
    *,
    *::before,
    *::after {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
  
    main {
      text-align: center;
      font-family: 'Nunito', sans-serif;
      font-weight: 300;
      line-height: 2;
      font-size: 24px;
      color: var(--color-text);
    }
  
    h1 {
      font-size: 2em;
      font-weight: 300;
      line-height: 2;
    }
  </style>
