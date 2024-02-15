<script>

  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Graph from '../components/Graph.svelte';
  import { Runtime, Inspector } from "@observablehq/runtime";
  import define from "@d3/versor-dragging";

  let data = [];
  let searchQuery = 'United States';
  let isLoading = true;
  let loadingError = null;
  let chartContainer;
  let countries = [];
  let matchingCountries = [];
  let filteredCountries = ["United States", "India", "China", "Russia", "Japan"];
  let country_hovered = -1;

  onMount(async () => {
    try {
      const res = await fetch('p3_energy_data.csv');
      const csv = await res.text();
      data = d3.csvParse(csv, (d) => {
        return {
          year: +d.year,
          country: d.country,
          biofuel: +d.biofuel_elec_per_capita,
          coal: +d.coal_elec_per_capita,
          gas: +d.gas_elec_per_capita,
          hydro: +d.hydro_elec_per_capita,
          nuclear: +d.nuclear_elec_per_capita,
          oil: +d.oil_elec_per_capita,
          solar: +d.solar_elec_per_capita,
          wind: +d.wind_elec_per_capita,
          fossil: +d.fossil_elec_per_capita,
          low_carbon: +d.low_carbon_elec_per_capita,
        };
      });
      countries = new Set(data.map(d => d.country));
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

  $: countries = Array.from(countries);
  $: matchingCountries = countries.filter(country =>
    country.toLowerCase().startsWith(searchQuery.toLowerCase())
  );

  $: filteredCountries = matchingCountries.slice(0, 5);

  function setDefault() {
    filteredCountries = ["United States", "India", "China", "Russia", "Japan"];
  }
  $: {
    // Check if searchQuery is an empty string, then call setDefault
    if (searchQuery === '') {
      setDefault();
    }
  }

  function setSearchQuery(country) {
    searchQuery = country;
  }

  function clearSearch() {
    searchQuery = '';
    setDefault(); // Reset filteredCountries when clearing the search
  }
</script>


<main>
  <h1>Elec Per Capita</h1>
  <!-- <form on:submit|preventDefault={add}> -->
    <input bind:value={searchQuery} placeholder="Search for a Country" on:input />
    <button on:click={clearSearch} class="clear-button">X</button>
    {#if !countries.includes(searchQuery)}
      <div>
        {#each filteredCountries as country}
        <button 
            on:click={() => setSearchQuery(country)}
            on:mouseover={(event) => { country_hovered = country;}}
            on:mouseout={(event) => { country_hovered = -1; }}
            class:active={country_hovered === country}
            >         
            {country}
          </button>
        {/each}
      </div>
    {/if}
  <!-- </form> -->

  {#if isLoading}
    <p>Loading...</p>
  {:else if loadingError}
    <p>{loadingError}</p>
  {:else}
    <Graph {data} {searchQuery}/>
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

    button {
      margin: 5px; /* Adjust the margin as needed */
      padding: 5px; /* Adjust the padding as needed */
      width: 150px; /* Adjust the width as needed */
      height: 60px; /* Adjust the height as needed */
      background-color: #FFC4EB; 
      color: #000000; 
      font-family: 'Nunito', sans-serif;
      font-size: 14px;
      vertical-align: middle;
    }
    .clear-button {
      background-color: #FE8787; 
      color: #000000;
      width: 22px; 
      height: 22px;
      font-size: 11px;
      /* line-height: -20px; */
      vertical-align: middle;
      border-radius: 10px;
      font-weight: bold;
      padding: 2px;
    }
    button.active {
      /* Styles for the active state go here */
      background-color: #DD6AB6;
      color: #000000;
    }
  </style>