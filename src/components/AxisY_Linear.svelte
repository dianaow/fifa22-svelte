<script>
  export let width;
  export let yScale;
  export let title;
  
  let yTicks = yScale.ticks(10);
</script>

<g class='axis y'>
  {#each yTicks as tick, index}
    <g class='tick' transform="translate(0, {yScale(tick)})">
      <line x1={0} x2={width} y1={0} y2={0} stroke={index === 0 ? '#8f8f8f' : '#e5e7eb'} /> 
      <text y={-3}>{index === yTicks.length - 1 ? `${tick} euros` : tick}</text>
    </g>
  {/each}
  <text class="axis-title" x={0} y={-36} dy="4" dominant-baseline="hanging" text-anchor="start">
    {title}
   </text>
</g>