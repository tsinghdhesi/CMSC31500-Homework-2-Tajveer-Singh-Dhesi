<script>
  import * as d3 from 'd3';

  export let data;
  export let fullData;
  export let filter;
  export let update;

  let margin = { top: 10, right: 30, bottom: 35, left: 55 };
  let width = 440;
  let height = 260;
  let chartW = width - margin.left - margin.right;
  let chartH = height - margin.top - margin.bottom;

  let brushLayer;
  let xAxisEl;
  let yAxisEl;

  const brush = d3.brushX()
    .extent([[0, 0], [chartW, chartH]])
    .on('brush', brushed)
    .on('end', brushended);

  function brushed(event) {
    if (event && event.selection) {
      filter = [xScale.invert(event.selection[0]), xScale.invert(event.selection[1])];
      update();
    }
  }

  function brushended(event) {
    if (event && !event.selection) {
      filter = [];
      update();
    }
  }

  $: xScale = d3.scaleLinear()
    .range([0, chartW])
    .domain(d3.extent(fullData.map(d => d.mag)));

  $: binData = d3.bin()
    .value(d => d.mag)
    .domain(xScale.domain())
    .thresholds(xScale.ticks(20));

  $: backgroundBins = binData(fullData);
  $: bins = binData(data);

  $: yScale = d3.scaleLinear()
    .range([chartH, 0])
    .domain([0, d3.max(backgroundBins, d => d.length)]);

  $: {
    d3.select(brushLayer)
      .call(brush);
    d3.select(xAxisEl)
      .call(d3.axisBottom(xScale).ticks(8));
    d3.select(yAxisEl)
      .call(d3.axisLeft(yScale).ticks(5));
  }
</script>

<svg {width} {height}>
  <g transform="translate({margin.left}, {margin.top})">
    {#each backgroundBins as d}
      <rect class="backgroundbar"
        x={xScale(d.x0)}
        y={yScale(d.length)}
        width={Math.max(0, xScale(d.x1) - xScale(d.x0) - 1)}
        height={chartH - yScale(d.length)} />
    {/each}
    {#each bins as d}
      <rect class="bar"
        x={xScale(d.x0)}
        y={yScale(d.length)}
        width={Math.max(0, xScale(d.x1) - xScale(d.x0) - 1)}
        height={chartH - yScale(d.length)} />
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
    stroke: white;
    stroke-width: 0.5px;
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
