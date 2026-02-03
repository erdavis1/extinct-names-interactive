<!--
  @component
  Generates an SVG scatter plot.
 -->
<script>
  import { getContext } from 'svelte';

  const { data, xGet, yGet, xScale, yScale } = getContext('LayerCake');

  /**
   * @typedef {Object} Props
   * @property {number} [r=5] - The circle's radius.
   * @property {string} [fill='#0cf'] - The circle's fill color.
   * @property {string} [stroke='#000'] - The circle's stroke color.
   * @property {number} [strokeWidth=0] - The circle's stroke width.
   * @property {(d: any) => boolean} [defined=() => true] - Whether a point should be rendered.
   */

  /** @type {Props} */
  let {
    r = 5,
    fill = '#0cf',
    stroke = '#eee8dc',
    strokeWidth = 0,
    defined = () => true
  } = $props();

  // Filter data reactively
  let filteredData = $derived($data.filter(d => defined(d)));

  // Band offsets (ordinal scales)
  let xOffset = $derived(
    typeof $xScale.bandwidth === 'function' ? $xScale.bandwidth() / 2 : 0
  );

  let yOffset = $derived(
    typeof $yScale.bandwidth === 'function' ? $yScale.bandwidth() / 2 : 0
  );
</script>

<g class="scatter-group">
  {#each filteredData as d}
    <circle
      cx={$xGet(d) + xOffset}
      cy={$yGet(d) + yOffset}
      {r}
      {fill}
      {stroke}
      stroke-width={strokeWidth}
    />
  {/each}
</g>
