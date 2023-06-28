<script>
  import AxisX from "$components/AxisX.svelte";
  import AxisY_Linear from "$components/AxisY_Linear.svelte";
  import Tooltip_Scatter from "$components/Tooltip_Scatter.svelte";

  import { scaleLinear, scaleOrdinal, scaleSqrt } from "d3-scale";
  import { extent, mean, rollups } from "d3-array";

  export let data, clubs
  let width = 400,
    height = 450;

  const margin = { top: 40, right: 30, bottom: 40, left: 0 };

  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  $: ageRollup = rollups(
    data,
    v => mean(v, d => +d.age),
    d => d.club_name
  )

  $: dataWageRollup = rollups(
    data,
    v => mean(v, d => d.wage_eur),
    d => d.club_name
  )

  $: dataValueRollup = rollups(
    data,
    v => mean(v, d => d.overall),
    d => d.club_name
  ).sort((a, b) => a[1] - b[1])

  $: scatterData = dataWageRollup.map((d, i) => {
    const clubName = d[0];
    const wage = d[1];
    const score = dataValueRollup.find(a => a[0] === clubName)[1];
    const age = ageRollup.find(a => a[0] === clubName)[1];
    const clubNameRelabel = clubs.clubsToHighlight.includes(clubName) ? clubName : "Other";

    return { wage, club_name: clubName, score, age, club_name_relabel: clubNameRelabel };
  });

  let colorScale = scaleOrdinal()
    .domain(clubs.clubsToHighlight.concat('Other')) 
    .range(clubs.colorRange);

  $: xScale = scaleLinear()
    .domain([64, 84])
    .range([0, innerWidth]);

  let yScale = scaleLinear()
    //.domain(extent(scatterData, d => d.wage))
    .domain([0, 180000])
    .range([innerHeight, 0]);

  $: radiusScale = scaleSqrt()
  .domain([22, 28])
  .range(window.innerWidth < 1440 ? [2, 6] : [3, 8]);

  let hoveredData;

  import { fly } from "svelte/transition";
</script>

<div
  class="chart-container"
  bind:clientWidth={width}
>
  <svg {width} {height} on:mouseleave={() => hoveredData = null}>
    <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
        <AxisY_Linear width={innerWidth} {yScale} title={'Average wage of players (in euros)'} tickNum=10/>
        <AxisX height={innerHeight} width={innerWidth} {xScale} title={'Average score of players'}/>
        {#each scatterData as d, index}
          <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
          <circle 
            in:fly={{ x: -10, opacity: 0, duration: 500 }}
            cx={xScale(d.score)}
            cy={yScale(d.wage)}
            r={radiusScale(d.age)}
            fill={colorScale(d.club_name_relabel)}
            stroke={hoveredData
              ? hoveredData === d
                ? "black"
                : "transparent"
              : "black"}
            opacity={hoveredData ? (hoveredData == d ? 1 : 0.45) : 1}
            on:mouseover={() => hoveredData = d}
            on:focus={() => hoveredData = d}
            tabindex="0"
          />
        {/each}
    </g>
  </svg>
  {#if hoveredData}
    <Tooltip_Scatter {xScale} {yScale} {colorScale} {width} data={hoveredData} />
  {/if}
</div>

<style>
  :global(.tick text, .axis-title) {
    font-weight: 400; /* How thick our text is */
    font-size: 12px; /* How big our text is */
    fill: hsla(212, 10%, 53%, 1); /* The color of our text */
  }

  .chart-container {
    position: relative;
  }

  circle {
    transition: r 300ms ease, opacity 500ms ease;
    cursor: pointer;
  }
</style>
