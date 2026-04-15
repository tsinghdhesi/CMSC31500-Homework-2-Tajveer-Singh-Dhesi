<script>
  import * as d3 from 'd3';

  export let data;
  export let fullData;
  export let selectedYears;
  export let update;

  let margin = { top: 10, right: 20, bottom: 40, left: 55 };
  let width = 440;
  let height = 260;
  let chartW = width - margin.left - margin.right;
  let chartH = height - margin.top - margin.bottom;

  let xAxisEl;
  let yAxisEl;

  $: allYears = [...new Set(fullData.map(d => d.year))].sort((a, b) => a - b);

  // Counts from filtered data
  $: yearCounts = (() => {
    const counts = {};
    for (let y of allYears) counts[y] = 0;
    for (let d of data) counts[d.year] = (counts[d.year] || 0) + 1;
    return counts;
  })();

  // Background counts from full data
  $: bgCounts = (() => {
    const counts = {};
    for (let y of allYears) counts[y] = 0;
    for (let d of fullData) counts[d.year] = (counts[d.year] || 0) + 1;
    return counts;
  })();

  $: xScale = d3.scaleBand()
    .domain(allYears)
    .range([0, chartW])
    .padding(0.15);

  $: yScale = d3.scaleLinear()
    .domain([0, d3.max(Object.values(bgCounts)) || 1])
    .range([chartH, 0]);

  $: hasSelection = selectedYears.length > 0;

  function toggleYear(year) {
    if (selectedYears.includes(year)) {
      selectedYears = selectedYears.filter(y => y !== year);
    } else {
      selectedYears = [...selectedYears, year];
    }
    update();
  }

  $: {
    if (xAxisEl) {
      d3.select(xAxisEl)
        .call(d3.axisBottom(xScale).tickValues(
          allYears.filter((_, i) => i % 5 === 0 || i === allYears.length - 1)
        ))
        .selectAll('text')
        .attr('transform', 'rotate(-40)')
        .style('text-anchor', 'end');
    }
    if (yAxisEl) {
      d3.select(yAxisEl).call(d3.axisLeft(yScale).ticks(5));
    }
  }
</script>

<svg {width} {height}>
  <g transform="translate({margin.left}, {margin.top})">
    {#each allYears as year}
      {@const count = yearCounts[year] || 0}
      {@const bgCount = bgCounts[year] || 0}
      {@const isSelected = selectedYears.includes(year)}

      <rect class="backgroundbar"
        x={xScale(year)}
        y={yScale(bgCount)}
        width={xScale.bandwidth()}
        height={chartH - yScale(bgCount)}
      />

      <rect
        class="bar"
        class:dimmed={hasSelection && !isSelected}
        class:selected={isSelected}
        x={xScale(year)}
        y={yScale(count)}
        width={xScale.bandwidth()}
        height={chartH - yScale(count)}
        on:click={() => toggleYear(year)}
        on:keydown={(e) => { if (e.key === 'Enter') toggleYear(year); }}
        role="button"
        tabindex="0"
      />
    {/each}
  </g>

  <g class="axis" transform="translate({margin.left}, {chartH + margin.top})"
     bind:this={xAxisEl} />
  <g class="axis" transform="translate({margin.left}, {margin.top})"
     bind:this={yAxisEl} />

  <text class="axis-label" x={margin.left + chartW / 2} y={height - 2} text-anchor="middle">
    Year
  </text>
  <text class="axis-label" x={12} y={margin.top + chartH / 2}
        text-anchor="middle" transform="rotate(-90, 12, {margin.top + chartH / 2})">
    Count
  </text>
</svg>

<style>
  .backgroundbar {
    fill: #ddd;
  }

  .bar {
    fill: var(--color-bar, #e85d26);
    cursor: pointer;
    transition: opacity 0.12s;
  }

  .bar:hover {
    fill: #c94e1e;
  }

  .bar.dimmed {
    opacity: 0.25;
  }

  .bar.selected {
    stroke: #333;
    stroke-width: 1.5px;
    opacity: 1;
  }

  .axis-label {
    font-size: 11px;
    fill: #666;
    font-weight: 500;
  }

  :global(.axis text) {
    font-size: 10px;
    fill: #666;
  }

  :global(.axis line, .axis path) {
    stroke: #ccc;
  }
</style>
