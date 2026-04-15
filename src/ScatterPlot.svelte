<script>
  import * as d3 from 'd3';

  export let data;
  export let fullData;
  export let filter;
  export let update;

  let margin = { top: 10, right: 20, bottom: 35, left: 55 };
  let width = 520;
  let height = 260;
  let chartW = width - margin.left - margin.right;
  let chartH = height - margin.top - margin.bottom;

  let brushLayer;
  let xAxisEl;
  let yAxisEl;

  let tooltipText = '';
  let tooltipX = 0;
  let tooltipY = 0;
  let showTooltip = false;

  const brush = d3.brush()
    .extent([[0, 0], [chartW, chartH]])
    .on('brush', brushed)
    .on('end', brushended);

  function brushed(event) {
    if (event && event.selection) {
      const [[x0, y0], [x1, y1]] = event.selection;
      filter = {
        mag: [xScale.invert(x0), xScale.invert(x1)],
        depth: [yScale.invert(y0), yScale.invert(y1)]
      };
      update();
    }
  }

  function brushended(event) {
    if (event && !event.selection) {
      filter = null;
      update();
    }
  }

  $: xScale = d3.scaleLinear()
    .domain(d3.extent(fullData.map(d => d.mag)))
    .range([0, chartW]);

  $: yScale = d3.scaleLinear()
    .domain([0, d3.max(fullData, d => d.depth) || 100])
    .range([0, chartH]);

  $: {
    d3.select(brushLayer)
      .call(brush);
    d3.select(xAxisEl)
      .call(d3.axisBottom(xScale).ticks(8));
    d3.select(yAxisEl)
      .call(d3.axisLeft(yScale).ticks(6));
  }

  function isInBrush(d) {
    if (!filter) return true;
    return d.mag >= filter.mag[0] && d.mag <= filter.mag[1]
        && d.depth >= filter.depth[0] && d.depth <= filter.depth[1];
  }

  function handleDotHover(event, d) {
    tooltipText = `M${d.mag.toFixed(1)} · ${d.depth.toFixed(1)} km · ${d.place || ''}`;
    tooltipX = event.offsetX + 12;
    tooltipY = event.offsetY - 10;
    showTooltip = true;
  }

  function handleDotLeave() {
    showTooltip = false;
  }
</script>

<div class="scatter-container">
  <svg {width} {height}>
    <g transform="translate({margin.left}, {margin.top})">
      {#each data as d}
        <circle
          class="dot"
          class:dimmed={!isInBrush(d)}
          cx={xScale(d.mag)}
          cy={yScale(d.depth)}
          r="2"
          on:mousemove={(e) => handleDotHover(e, d)}
          on:mouseleave={handleDotLeave}
          role="img"
          aria-label="earthquake"
        />
      {/each}
    </g>

    <g transform="translate({margin.left}, {margin.top})"
       bind:this={brushLayer} />

    <g class="axis" transform="translate({margin.left}, {chartH + margin.top})"
       bind:this={xAxisEl} />
    <g class="axis" transform="translate({margin.left}, {margin.top})"
       bind:this={yAxisEl} />

    <text class="axis-label" x={margin.left + chartW / 2} y={height - 2} text-anchor="middle">
      Magnitude
    </text>
    <text class="axis-label" x={14} y={margin.top + chartH / 2}
          text-anchor="middle" transform="rotate(-90, 14, {margin.top + chartH / 2})">
      Depth (km)
    </text>
  </svg>

  {#if showTooltip}
    <div class="tooltip" style="left:{tooltipX}px; top:{tooltipY}px;">
      {tooltipText}
    </div>
  {/if}
</div>

<style>
  .scatter-container {
    position: relative;
  }

  .dot {
    fill: var(--color-dot, #e85d26);
    fill-opacity: 0.35;
    stroke: none;
  }

  .dot.dimmed {
    fill-opacity: 0.06;
    fill: #aaa;
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

  .tooltip {
    position: absolute;
    background: rgba(0, 0, 0, 0.82);
    color: #fff;
    padding: 4px 8px;
    border-radius: 4px;
    font-size: 0.72rem;
    pointer-events: none;
    white-space: nowrap;
    z-index: 10;
    max-width: 320px;
    overflow: hidden;
    text-overflow: ellipsis;
  }
</style>
