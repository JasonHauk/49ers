<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Info from '../components/Info.svelte';
  import Graph from '../components/Graph.svelte';

  let data = [];

  onMount(async () => {
    const res = await fetch(
      'https://nyc3.digitaloceanspaces.com/owid-public/data/energy/owid-energy-data.csv',
    );
    const csv = await res.text();
    await d3.csvParse(csv, (d, i, columns) => {
      for (let i = 1; i < 13; ++i) {
        // pivot longer
        // console.log(+d[columns[i]])
        data.push({
          country: new Date(Date.UTC(d.Year, i - 1, 1)),
          value: +d[columns[i]],
        });
      }
    });
    data = data;
    console.log(data)
  });
</script>

<main>
  <h1>Global Temperature Trends</h1>
  <Graph {data} />
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
