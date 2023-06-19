<script>
  export let xScale;
  export let width;
  export let height;
  export let title;

  $: ticks = xScale.ticks(8);
</script>

<g class="axis x">
  {#each ticks as tick}
    <g class="tick" transform="translate({xScale(tick)}, {height+20})">
      <text x="3" y={0}>{tick}</text>
      <line x1="0" x2="0" y1="0" y2={height} stroke="hsla(215, 15%, 91%, 1)" />
    </g>
  {/each}
  <text class="axis-title" x={width} y={height+24} dy="4" dominant-baseline="hanging" text-anchor="end">
   {title}
  </text>
</g>
