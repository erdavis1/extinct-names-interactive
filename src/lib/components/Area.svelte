<!--
  @component
  Generates an SVG area shape.
-->
<script>
  import { getContext } from 'svelte';

  const { data, xGet, yGet, xScale, yScale, extents } = getContext('LayerCake');

  /**
   * @typedef {Object} Props
   * @property {string} [fill='#ab00d6'] - The area fill color
   * @property {(d: object) => boolean} [defined] - Function to skip points
   * @property {number} [opacity=1] - The area opacity
   */

  /** @type {Props} */
  let { fill = '#ab00d6', defined = () => true, opacity = 1 } = $props();

  // build path with proper baseline, breaking at undefined points
  let path = $derived.by(() => {
    const yRange = $yScale.range(); // usually [height, 0]
    const y0 = yRange[0]; // baseline (bottom)

    let d = '';
    let started = false;

    for (const point of $data) {
      if (!defined(point)) {
        started = false;
        continue;
      }

      const x = $xGet(point);
      const y = $yGet(point);

      if (!started) {
        // start new subpath at baseline
        d += `M${x},${y0} L${x},${y}`;
        started = true;
      } else {
        d += `L${x},${y}`;
      }
    }

    // close subpaths to baseline
    // we need to go back along x to baseline in reverse order
    let subpaths = d.split('M').filter(Boolean).map(sp => {
      // sp = "x0,y0 Lx1,y1 ..."
      const coords = sp.trim().split('L').map(c => c.split(',').map(Number));
      const last = coords[coords.length - 1];
      const first = coords[0];
      let closing = `L${last[0]},${y0} L${first[0]},${y0} Z`;
      return 'M' + sp + closing;
    });

    return subpaths.join('');
  });
</script>

<path class="path-area" d={path} {fill} opacity={opacity}></path>

<style>
  .path-area {
    stroke: none;
    stroke-width: 0;
  }
</style>
