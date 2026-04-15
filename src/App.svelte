<script>
  import * as d3 from 'd3';
  import { onMount } from 'svelte';
  import ChoroplethMap from './ChoroplethMap.svelte';
  import Histogram from './Histogram.svelte';
  import ScatterPlot from './ScatterPlot.svelte';
  import BarChart from './BarChart.svelte';

  let enrichedData = [];
  let geoData = null;
  let loading = true;

  // Filter states for each view
  let selectedStates = [];
  let magBrush = [];
  let scatterBrush = null;
  let selectedYears = [];

  // Filtered data arrays — recomputed by updateData()
  let dataForMap = [];
  let dataForHist = [];
  let dataForScatter = [];
  let dataForBar = [];

  onMount(async function () {
    const [csv, geo] = await Promise.all([
      d3.csv('earthquakes.csv', (d) => ({
        time: d.time,
        latitude: +d.latitude,
        longitude: +d.longitude,
        depth: +d.depth,
        mag: +d.mag,
        place: d.place,
        year: new Date(d.time).getFullYear()
      })),
      d3.json('india_states.geojson')
    ]);

    geoData = geo;

    // assign each earthquake to a state
    for (let eq of csv) {
      let assigned = null;
      for (let feature of geo.features) {
        if (d3.geoContains(feature, [eq.longitude, eq.latitude])) {
          assigned = feature.properties.NAME_1;
          break;
        }
      }
      eq.state = assigned;
    }

    // Only keep earthquakes that fall within an Indian state
    enrichedData = csv.filter(d => d.state !== null);
    loading = false;
    updateData();
  });

  function passesStateFilter(d) {
    if (selectedStates.length === 0) return true;
    return selectedStates.includes(d.state);
  }

  function passesMagFilter(d) {
    if (magBrush.length === 0) return true;
    return d.mag >= magBrush[0] && d.mag <= magBrush[1];
  }

  function passesScatterFilter(d) {
    if (!scatterBrush) return true;
    return d.mag >= scatterBrush.mag[0] && d.mag <= scatterBrush.mag[1]
        && d.depth >= scatterBrush.depth[0] && d.depth <= scatterBrush.depth[1];
  }

  function passesYearFilter(d) {
    if (selectedYears.length === 0) return true;
    return selectedYears.includes(d.year);
  }

  function updateData() {
    dataForMap = enrichedData.filter(d =>
      passesMagFilter(d) && passesScatterFilter(d) && passesYearFilter(d)
    );
    dataForHist = enrichedData.filter(d =>
      passesStateFilter(d) && passesScatterFilter(d) && passesYearFilter(d)
    );
    dataForScatter = enrichedData.filter(d =>
      passesStateFilter(d) && passesMagFilter(d) && passesYearFilter(d)
    );
    dataForBar = enrichedData.filter(d =>
      passesStateFilter(d) && passesMagFilter(d) && passesScatterFilter(d)
    );
  }

  $: hasActiveFilter = selectedStates.length > 0 || magBrush.length > 0
                     || scatterBrush !== null || selectedYears.length > 0;

  function resetAll() {
    selectedStates = [];
    magBrush = [];
    scatterBrush = null;
    selectedYears = [];
    updateData();
  }
</script>

<main>
  <header>
    <div class="title-row">
      <div>
        <h1>India Earthquake Explorer</h1>
        <p class="subtitle">USGS Catalog · M2.5+ · 2000–2025 · {enrichedData.length.toLocaleString()} events</p>
      </div>
      {#if hasActiveFilter}
        <button class="reset-btn" on:click={resetAll}>✕ Reset All Filters</button>
      {/if}
    </div>
  </header>

  {#if loading}
    <div class="loading"><p>Loading earthquake data and computing point-in-polygon assignments…</p></div>
  {:else}
    <div class="grid">
      <div class="panel map-panel">
        <h2>Earthquake Count by State</h2>
        <ChoroplethMap
          data={dataForMap}
          fullData={enrichedData}
          {geoData}
          bind:selectedStates={selectedStates}
          update={updateData}
        />
      </div>
      <div class="panel">
        <h2>Magnitude Distribution</h2>
        <Histogram
          data={dataForHist}
          fullData={enrichedData}
          bind:filter={magBrush}
          update={updateData}
        />
      </div>
      <div class="panel">
        <h2>Depth vs. Magnitude</h2>
        <ScatterPlot
          data={dataForScatter}
          fullData={enrichedData}
          bind:filter={scatterBrush}
          update={updateData}
        />
      </div>
      <div class="panel">
        <h2>Earthquakes by Year</h2>
        <BarChart
          data={dataForBar}
          fullData={enrichedData}
          bind:selectedYears={selectedYears}
          update={updateData}
        />
      </div>
    </div>
  {/if}
</main>

<style>
  header { margin-bottom: 1rem; }

  .title-row {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    flex-wrap: wrap;
    gap: 0.75rem;
  }

  h1 { font-size: 1.6rem; font-weight: 700; letter-spacing: -0.02em; }

  .subtitle { color: var(--color-muted); font-size: 0.85rem; margin-top: 0.15rem; }

  .reset-btn {
    background: var(--color-accent);
    color: white;
    border: none;
    padding: 0.45rem 1rem;
    border-radius: 6px;
    font-size: 0.82rem;
    font-weight: 600;
    cursor: pointer;
    transition: background 0.15s;
    white-space: nowrap;
  }

  .reset-btn:hover { background: #c94e1e; }

  .loading {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 60vh;
    color: var(--color-muted);
  }

  .grid {
    display: grid;
    grid-template-columns: 1.2fr 1fr;
    grid-template-rows: auto auto;
    gap: 0.75rem;
  }

  .panel {
    background: var(--color-panel);
    border: 1px solid var(--color-border);
    border-radius: 8px;
    padding: 1rem;
    overflow: hidden;
  }

  .panel h2 {
    font-size: 0.85rem;
    font-weight: 600;
    color: var(--color-muted);
    text-transform: uppercase;
    letter-spacing: 0.05em;
    margin-bottom: 0.5rem;
  }

  @media (max-width: 900px) {
    .grid { grid-template-columns: 1fr; }
  }
</style>
