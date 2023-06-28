<script>
  import { onMount, afterUpdate } from "svelte";
  import AxisX from "$components/AxisX.svelte";
  import AxisY_Categorical from "$components/AxisY_Categorical.svelte";
  import Legend from "$components/Legend.svelte";
  import Tooltip_Beeswarm from "$components/Tooltip_Beeswarm.svelte";

  import { forceSimulation, forceX, forceY, forceCollide } from "d3-force";
  import { scaleLinear, scaleBand, scaleOrdinal, scaleSqrt } from "d3-scale";
  import { mean, rollups } from "d3-array";

  export let data, clubs
  let width = 400,
    height = 430;
  const margin = { top: 0, right: 0, left: 0, bottom: 40 };
  
  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  // Generate the average for each club, so that we can sort according to that
  const categories = ['pace', 'shooting', 'passing', 'dribbling', 'defending', 'physic']

  let colorScale = scaleOrdinal()
    .domain(clubs.clubsToHighlight.concat('Other')) 
    .range(clubs.colorRange);

  $: radiusScale = scaleSqrt()
    .domain([22, 28])
    .range(window.innerWidth < 1440 ? [2, 6] : [3, 8]);

  $: xScale = scaleLinear()
    .domain([40, 70])
    .range([0, innerWidth]);

  let yScale = scaleBand()
    .domain(categories)
    .range([innerHeight, 0])
    .paddingOuter(0.5);

  let nodes = [];

  $: updateBeeswarmData(data, innerWidth);

  function updateBeeswarmData(data, innerWidth){
    let newData = categories.map(cat => {
      return data.map(d => {
        const club_name = d['club_name']
        const category = cat
        const value = +d[cat]
        return { value, club_name, category};
      })
    });

    let ageRollup = rollups(
      data,
      v => mean(v, d => +d.age),
      d => d.club_name
    )

    let newDataRollup = rollups(
        newData.flat(),
        v => mean(v, d => d.value),
        d => d.category,
        d => d.club_name
      )

    let beeswarmData = newDataRollup.map((d, i) => {
      return d[1].map((el,i) => {
        const ageRollupD = ageRollup.find(a => a[0] === el[0])
        const category = d[0]
        const clubName = el[0]
        const value = el[1]
        const age = ageRollupD ? ageRollupD[1] : null
        const clubNameRelabel = clubs.clubsToHighlight.includes(clubName) ? clubName : "Other"
        if(age){
          return { category, club_name: clubName, value, age, club_name_relabel: clubNameRelabel, x: xScale(value), y: groupByCategory ? yScale(category) : innerHeight / 2}
        } else {
          return
        }
      })
    });

    radiusScale.range(window.innerWidth < 1440 ? [2, 6] : [3, 8]);

    xScale.range([0, innerWidth]);

    let simulation = forceSimulation(beeswarmData.flat())
      .force(
        "x",
        forceX()
          .x(d => xScale(d.value))
          .strength(0.9)
      )
      .force(
        "y",
        forceY()
          .y(d => (groupByCategory ? yScale(d.category) : innerHeight / 2))
          .strength(0.2)
      )
      .force("collide", forceCollide().radius(d => radiusScale(d.age) * 1.4))
      .alpha(0.3) // [0, 1] The rate at which the simulation finishes. You should increase this if you want a faster simulation, or decrease it if you want more "movement" in the simulation.
      .alphaDecay(0.0005) // [0, 1] The rate at which the simulation alpha approaches 0. you should decrease this if your bubbles are not completing their transitions between simulation states.
      .restart();

    simulation.on("tick", () => {
      nodes = simulation.nodes();
    });

  }

  // Highlight hovered circles and deemphasize others
  // If a category is hovered (from Legend.svelte), highlight relevant circles
  let hovered;
  import { fade } from "svelte/transition";

  // Boolean for whether the circles are grouped
  let groupByCategory = true;
</script>

<Legend {colorScale} />
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div
  class="chart-container"
  bind:clientWidth={width}
  on:click={() => {
    groupByCategory = !groupByCategory;
    hovered = null;
  }}
>
  <svg {width} {height}>
    <!-- Reference line -->
    {#if hovered}
      <line
        transition:fade
        x1={hovered.x}
        x2={hovered.x}
        y1={height - margin.bottom}
        y2={hovered.y + margin.top + radiusScale(hovered.age)}
        stroke={colorScale(hovered.club_name_relabel)}
        stroke-width="2"
      />
    {/if}
    <g
      class="inner-chart"
      transform="translate({margin.left}, {margin.top})"
      on:mouseleave={() => (hovered = null)}
    >
      <AxisX {xScale} height={innerHeight} width={innerWidth} title={'Average score of all players in a club'}/>
      <AxisY_Categorical {yScale} {groupByCategory}/>
      {#each nodes as node, i}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
        <circle       
          in:fade={{ delay: 200 + 10 * i, duration: 400 }}
          cx={node.x}
          cy={node.y}
          r={radiusScale(node.age)}
          fill={colorScale(node.club_name_relabel)}
          title={node.club_name}
          opacity={hovered
            ? hovered === node 
              ? 1
              : 0.45
            : 1}
          stroke={hovered
            ? hovered === node
              ? "black"
              : "transparent"
            : "black"}
          on:mouseover={() => (hovered = node)}
          on:focus={() => (hovered = node)}
          tabindex="0"
          on:click={(event) => {
            event.stopImmediatePropagation();
          }}
        />
      {/each}
    </g>
  </svg>
  {#if hovered}
    <Tooltip_Beeswarm data={hovered} {colorScale} {width} />
  {/if}
</div>

<style>
  :global(*) {
    font-family: Inter, -apple-system, system-ui;
    -moz-osx-font-smoothing: grayscale;
  }

  :global(.tick text, .axis-title) {
    font-size: 12px; /* How big our text is */
    font-weight: 400; /* How bold our text is */
    fill: hsla(212, 10%, 53%, 1); /* The color of our text */
    user-select: none; /* Prevents text from being selected */
  }

  circle {
    transition: stroke 300ms ease, opacity 300ms ease;
    cursor: pointer;
  }
</style>
