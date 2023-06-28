<script>
  import { max, min, mean, rollups } from "d3-array";
  import { scaleLinear, scaleOrdinal } from "d3-scale";
  import { line } from "d3-shape";
  import AxisX from "$components/AxisX.svelte";
  import AxisY_Linear from "$components/AxisY_Linear.svelte";
  import HoverEvents from "$components/HoverEvents.svelte";

  export let data, clubs, hoveredDate

  const margin = { top: 30, right: 30, bottom: 30, left: 30 };

  let height = 150;
  let width = 800;

  let innerHeight = height - margin.top - margin.bottom;
  $: innerWidth = width - margin.left - margin.right;

  const lineData = rollups(
    data.filter(d => clubs.clubsToHighlight.includes(d.club_name)),
    v => mean(v, d => d.overall),
    d => d.club_name,
    d => d.year
  )

  const yScale = scaleLinear()
    .domain([70, 80])
    .range([innerHeight, 0]);

  const minDate = 2015
  const maxDate = 2022

  $: xScale = scaleLinear()
    .domain([minDate, maxDate])
    .range([0, innerWidth]);

  let colorScale = scaleOrdinal()
    .domain(clubs.clubsToHighlight.concat('Other')) 
    .range(clubs.colorRange);

  $: lineGenerator = line()
      .x((d) => xScale(d[0]))
      .y((d) => yScale(d[1]));
      
</script>

<div class="chart-container">
  <svg
    {width}
    {height}
    aria-labelledby="chart-title"
    aria-describedby="chart-description"
    role="img"
  >
    <g transform="translate({margin.left} {margin.top})">
      <AxisX
        height={innerHeight}
        width={innerWidth} 
        {xScale}
      />
      <AxisY_Linear 
        width={innerWidth} 
        {yScale} 
        title={'Average score of all players'}
        tickNum=3
      />
      {#each lineData as d, index}
        <path
          d={lineGenerator(d[1])}
          stroke={colorScale(clubs.clubsToHighlight.includes(d[0]) ? d[0] : "Other")}
          fill="transparent"
          stroke-width="2"
        />
      {/each}
      {#if hoveredDate}
        <HoverEvents
          width={innerWidth}
          height={innerHeight}
          {xScale}
          {margin}
          {maxDate}
          bind:hoveredDate
        />
      {/if}
    </g>
  </svg>
</div>