<!--
  @component
  Adds text annotations that get their x and y placement using the `xScale` and `yScale`.
 -->
<script>
  import { getContext } from 'svelte';

  const { xGet, yGet, percentRange } = getContext('LayerCake');
  /**
   * @typedef {Object} Annotation
   * @property {string} text - The text content of the annotation
   * @property {string} subtext - The subtext content of the annotation
   * @property {number} [dx] - Optional horizontal pixel offset
   * @property {number} [dy] - Optional vertical pixel offset
   * @property {number} [top] - CSS top position in pixels
   * @property {number} [right] - CSS right position in pixels
   * @property {number} [bottom] - CSS bottom position in pixels
   * @property {number} [left] - CSS left position in pixels

   * @description Additional dynamic properties can be added using data keys (e.g., [xKey]: value, [yKey]: value) for positioning based on chart data dimensions
   */

  /**
   * @typedef {Object} Props
   * @property {Array<Annotation>} annotations - A list of annotation objects.
   * @property {Function} [getText] - An accessor function to get the field to display.
   * @property {Function} [getSubText] - An accessor function to get the field to display.
   * @property {boolean} [pr] - If `true` will set the `top` and `left` CSS positions to percentages instead of pixels.
* @property {string} [color='#ff0000'] - The text color
   */
    
  /** @type {Props} */
  let { annotations,
    getText = d => d.text, 
    getSubText = d => d.subtext,
    color='#000000',
     pr = $percentRange } = $props();

  let units = $derived(pr === true ? '%' : 'px');



</script>

<div class="layercake-annotations">

  {#each annotations as d, i}
    {#if d.plot === true}
        <div
            class="layercake-annotation"
            data-id={i}
            style="color: {d.color}; 
            left: calc({$xGet(d)}{units} + {d.dx || 0}px); 
            top: calc({$yGet(d)}{units} + {d.dy || 0}px);"
        >
        {getText(d)}
        <div class = 'subtext'>
        {getSubText(d)}
        </div>
        </div>
    {/if}
  {/each}

</div>

<style>
  .layercake-annotation {
    position: absolute;
     font-family: 'Libre Franklin', sans-serif; 
     font-weight: 500;
    font-size: 0.8em;
     text-shadow: 0 0px 2px rgb(238 232 220 / 90%), 0 0px 4px rgb(238 232 220 / 90%), 0 0px 4px rgb(238 232 220 / 90%), 0 0px 3px rgb(238 232 220 / 90%), 0 0px 3px rgb(238 232 220 / 90%);
   
   }

    .subtext {
    font-weight: 400;
    }
</style>