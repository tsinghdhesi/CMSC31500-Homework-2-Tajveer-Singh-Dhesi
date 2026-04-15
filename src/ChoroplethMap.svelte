<script>
  import * as d3 from 'd3';

  export let data;       
  export let fullData;   
  export let geoData;    
  export let selectedStates;
  export let update;

  let width = 520;
  let height = 560;

  let tooltipText = '';
  let tooltipX = 0;
  let tooltipY = 0;
  let showTooltip = false;

  // Exclude far islands from projection fitting so mainland fills the panel
  const excludeFromFit = ['Andaman and Nicobar', 'Lakshadweep'];

  $: mainlandGeo = geoData ? {
    type: 'FeatureCollection',
    features: geoData.features.filter(f => !excludeFromFit.includes(f.properties.NAME_1))
  } : null;

  $: projection = mainlandGeo
    ? d3.geoMercator().fitSize([width, height], mainlandGeo)
    : null;

  $: path = projection ? d3.geoPath().projection(projection) : null;

  // Counts from filtered data for coloring
  $: stateCounts = (() => {
    const counts = {};
    if (geoData) {
      for (let f of geoData.features) counts[f.properties.NAME_1] = 0;
    }
    for (let d of data) {
      if (d.state) counts[d.state] = (counts[d.state] || 0) + 1;
    }
    return counts;
  })();

  // Counts from full data for color scale
  $: fullStateCounts = (() => {
    const counts = {};
    if (geoData) {
      for (let f of geoData.features) counts[f.properties.NAME_1] = 0;
    }
    for (let d of fullData) {
      if (d.state) counts[d.state] = (counts[d.state] || 0) + 1;
    }
    return counts;
  })();

  $: maxCount = Math.max(1, d3.max(Object.values(fullStateCounts)) || 1);

  $: colorScale = d3.scaleSequential(d3.interpolateYlOrRd)
    .domain([0, maxCount]);

  $: hasSelection = selectedStates.length > 0;

  function toggleState(name) {
    if (selectedStates.includes(name)) {
      selectedStates = selectedStates.filter(s => s !== name);
    } else {
      selectedStates = [...selectedStates, name];
    }
    update();
  }

  function handleMouseMove(event, feature) {
    const name = feature.properties.NAME_1;
    const count = stateCounts[name] || 0;
    tooltipText = `${name}: ${count.toLocaleString()} earthquakes`;
    // Use pageX/pageY relative to the container
    const rect = event.currentTarget.closest('.map-container').getBoundingClientRect();
    tooltipX = event.clientX - rect.left + 12;
    tooltipY = event.clientY - rect.top - 10;
    showTooltip = true;
  }

  function handleMouseLeave() {
    showTooltip = false;
  }

  $: legendSteps = 6;
  $: legendValues = d3.range(legendSteps + 1).map(i => Math.round(maxCount * i / legendSteps));
</script>

<div class="map-container">
  <svg {width} {height}>
    {#if geoData && path}
      {#each geoData.features as feature}
        {@const name = feature.properties.NAME_1}
        {@const count = stateCounts[name] || 0}
        {@const isSelected = selectedStates.includes(name)}
        <path
          style="fill: {colorScale(count)};"
          class:dimmed={hasSelection && !isSelected}
          class:selected={isSelected}
          d={path(feature)}
          on:click={() => toggleState(name)}
          on:keydown={(e) => { if (e.key === 'Enter') toggleState(name); }}
          on:mousemove={(e) => handleMouseMove(e, feature)}
          on:mouseleave={handleMouseLeave}
          role="button"
          tabindex="0"
        />
      {/each}

      {#each geoData.features as feature}
        <path class="geooverlay" d={path(feature)} />
      {/each}

      <g transform="translate({width - 55}, 20)">
        <text class="legend-title" x="6" y="-8" text-anchor="middle">Count</text>
        {#each legendValues as val, i}
          <rect
            x="0" y={i * 16} width="12" height="16"
            fill={colorScale(val)}
            stroke="#999" stroke-width="0.3"
          />
          <text class="legend-label" x="16" y={i * 16 + 12}>{val}</text>
        {/each}
      </g>
    {/if}
  </svg>

  {#if showTooltip}
    <div class="tooltip" style="left:{tooltipX}px; top:{tooltipY}px;">
      {tooltipText}
    </div>
  {/if}
</div>

<style>
  .map-container {
    position: relative;
  }

  /* Layer 2: always-visible borders (matches demo .geooverlay) */
  .geooverlay {
    stroke: black;
    stroke-width: 1px;
    fill: none;
    pointer-events: none;
  }

  /* Selection styling on the fill layer */
  .dimmed {
    opacity: 0.3;
  }

  .selected {
    stroke: var(--color-accent, #e85d26);
    stroke-width: 2.5px;
    opacity: 1;
  }

  .legend-title {
    font-size: 10px;
    font-weight: 600;
    fill: #555;
  }

  .legend-label {
    font-size: 9px;
    fill: #555;
  }

  .tooltip {
    position: absolute;
    background: rgba(0, 0, 0, 0.82);
    color: #fff;
    padding: 4px 8px;
    border-radius: 4px;
    font-size: 0.75rem;
    pointer-events: none;
    white-space: nowrap;
    z-index: 10;
  }
</style>
