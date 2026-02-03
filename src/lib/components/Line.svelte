<!--
  @component
  Generates an SVG line shape.
-->
<script>
  import { getContext } from 'svelte';

  const { data, xGet, yGet } = getContext('LayerCake');

  /**
   * @typedef {Object} Props
   * @property {string} [stroke='#ab00d6'] - The line color
   * @property {(d: object) => boolean} [defined] - Function to skip points
   * @property {boolean} [dashed=false] - Toggle dashed stroke
   */

  /** @type {Props} */
  let { stroke = '#ab00d6', defined = () => true, dashed = false } = $props();

  // build path string with breaks for undefined points
  let path = $derived.by(() => {
    let d = '';
    let started = false;

    for (const point of $data) {
      if (!defined(point)) {
        started = false; // break the path
        continue;
      }

      const x = $xGet(point);
      const y = $yGet(point);

      if (!started) {
        d += `M${x},${y}`; // start new subpath
        started = true;
      } else {
        d += `L${x},${y}`;
      }
    }

    return d;
  });
</script>

<path 
  class="path-line"  stroke-dasharray={dashed ? '4 6' : undefined}
  d={path} 
  {stroke}
  
>
</path>

<style>
  .path-line {
    fill: none;
    stroke-linejoin: round;
    stroke-linecap: round;
    stroke-width: 3.5;
  }
</style>
