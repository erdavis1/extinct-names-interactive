<script>
  export const prerender = true;
import { LayerCake, Svg,Html } from 'layercake';
 import { derived } from 'svelte/store';
 import {onMount, tick} from 'svelte';

//components
import Line from '$lib/components/Line.svelte';
import Area from '$lib/components/Area.svelte';
import AxisX from '$lib/components/AxisX.svelte';
import AxisY from '$lib/components/AxisY.svelte';
import Point from '$lib/components/Point.svelte';
import Annotation from '$lib/components/Annotation.svelte';
import Select from '$lib/components/Select.svelte';

//data
import rawData from '../data/data.json';
const xKey = 'year';
const yKey = 'n_surviving';

//filter list of names
const nameOptions= 
  [...new Set(rawData.map(d => d.name))]
  .sort((a, b) => a.localeCompare(b))
  .map(name => ({ value: name, label: name, group: name[0].toUpperCase()}));

let selectedValue = $state({ value: 'Gary', label: 'Gary',group: 'G' });
const groupBy = (item) => item.group;

//selected name to show
let filterName = $derived(selectedValue.value);

let filteredData = $derived(
  rawData.filter(
    d => d.name === filterName 
  )
);


// maxY: the maximum y value for filteredData where x <= 2024
let maxY = $derived(
  Math.max(
    ...filteredData
      .filter(d => d[xKey] <= 2024)
      .map(d => d[yKey])
  )
);


// plotData: add series and flag fields
let plotData = $derived(
  filteredData.map(d => ({
    ...d,
    series: d[xKey] <= 2024 ? 'historical' : 'future',
    flag: d[yKey] === maxY ? 'max' : null
  }))
);

function formatNumber(d) {
  if (d >= 1e6) {
    const v = d / 1e6;
    return (Number.isInteger(v) ? v : v.toFixed(1)) + 'm';
  }
  if (d >= 1e3) {
    const v = d / 1e3;
    return (Number.isInteger(v) ? v : v.toFixed(0)) + 'k';
  }

  const v = (d / 100).toFixed(0)*100
  return v
}  

//annotations
let maxPoint = $derived(filteredData.find(d => d[yKey] === maxY));
let maxYear = $derived(maxPoint ? maxPoint[xKey] : null);
let currentPoint = $derived((filteredData.find(d => d[xKey] === 2024) || {})[yKey]);
let futurePoint = $derived((filteredData.find(d => d[xKey] === 2050) || {})[yKey]);

let annotations = $derived(() => [
    {
        text: maxYear,
        subtext: formatNumber(maxY),
        [xKey]: maxYear,
        [yKey]: maxY,
        dx: 0,
        dy: -35,
        color: '#7f3e3a',
        plot: true
    },
    {
        text: '2024',
        subtext: formatNumber(currentPoint),
        [xKey]: 2024,
        [yKey]: currentPoint,
        dx: 10,
        dy: -20,
        color: '#7f3e3a',
        plot: maxYear <= 2015
    },
    {
        text: '2050',
        subtext: formatNumber(futurePoint),
        [xKey]: 2050,
        [yKey]: futurePoint,
        dx: 10,
        dy: -15,
        color: '#231f20',
        plot: true
    }
]);

//resize the select dropdown
let selectWidth = $state('auto');
let selectContainer = $state(null);
let measureEl = $state(null);

function updateWidth() {
  if (!measureEl) return;

  const width = measureEl.width;
  selectWidth = `${width + 12}px`; // small breathing room
  console.log('updated width to', selectWidth);
}

onMount(async () => {
  await tick();
   updateWidth();
});

$effect(() => {
  selectedValue; 
  tick().then(updateWidth);
});

</script>


<div class="chart-container">

<div class = 'header'>
  <p class = "header-minor">ESTIMATED NUMBER OF</p>
  <div class="header-main">
    PEOPLE NAMED&nbsp;
      <span class="headline-select" bind:this={selectContainer} style="width: {selectWidth};">
        <Select
          items={nameOptions}
          itemId="value"
        groupBy={groupBy}
          bind:value={selectedValue}
        />
    </span>

    <!-- invisible measuring element -->
    <span class="select-measure" bind:this={measureEl}>
      {selectedValue?.label ?? ' '}
    </span>
  </div>

  <p class = "header-minor">ALIVE IN THE UNITED STATES</p>
</div>

  <LayerCake
      padding={{ top: 8, right: 40, bottom: 20, left: 35 }}
      x={xKey}
      y={yKey}
      yDomain={[0, null]}
      data={plotData}
    >
    <Svg>
      <!-- Y axis is at the bottom-->
      <AxisY 
        ticks = {10}
        gridlines={true}
        tickMarks = {true}
        tickMarkLength={10}
        format={formatNumber}
      />

      <!-- Lines and areas -->
      <!-- Historical -->
      <Line 
        defined={d => d.series === 'historical'}  
        stroke = '#AA5F54'
        dashed = {false}
      />
      <Area 
        defined={d => d.series === 'historical'}  
        fill = "#C97B6A"
        opacity={.65}
      />
      <!-- Projected -->
      <Line 
        defined={d => d.series === 'future'  || d[xKey] === 2024} 
        stroke = '#6b6b66'
        dashed = {true}
      />
      <Area 
        defined={d => d.series === 'future'  || d[xKey] === 2024}  
        fill = "#9b9a90"
        opacity={.45}
      />

      <!--Max point, and current point-->
      <Point 
          defined={d => d.flag === 'max' || (d[xKey] === 2024 && maxYear <= 2015)} 
          r={5} 
          fill="#7d3d38"  strokeWidth = 2
      />
      <Point 
          defined={d => d[xKey] === 2050} 
          r={5} 
          fill="#4d4d49"  strokeWidth = 2
      />
    
      <!-- X axis on top to plot over the areas-->
      <AxisX 
        xBaseline = {true}
        yBaseline = {true}
        gridlines={false}
        tickMarks = {true}
        tickMarkLength={10}
        ticks={[1900, 1925, 1950, 1975, 2000, 2025, 2050]}
      />
    </Svg>

    <Html>
        <Annotation annotations={annotations()} />
    </Html>
  </LayerCake>

</div>


<style>
:global(body) {
  background-color: #eee8dc;
  
}

.chart-container {
  width: 100%;
  height: 350px;
  font-size: clamp(14px, 2.5vw, 16px);
}

.header {
  padding-bottom: 3rem;
}

.header-minor {
  font-family: 'Libre Caslon Text', serif;
  font-weight: 300;
  font-size: 0.7em;
  margin: 0;
  color: #4a4645;
  text-align: center;
}

 .header-main {
  display: flex;
  align-items: center; 
  justify-content: center; 
  font-family: 'Libre Caslon Text', serif;
  font-weight: 400;
  font-size: 1.2em;
  margin: 0;
  color: #383634;

}

 .headline-select {
    --font-family: 'Libre Caslon Text', serif;
    --font-size: 1em;
    --input-color: #383634;
    --item-color: #383634;
    --font-weight: 400;

    --value-container-padding: 2px 8px 2px 8px;
    --selected-item-padding: 0;
    --height: 28px;       
    --item-height: 28px; 
    --placeholder-color: #8a8785;
    --icons-color: #8a8785;

    display: inline-block;
    min-width: 7ch;
    max-width: 100ch; 
    text-transform: uppercase;

 }


.select-measure {
  position: absolute;
  visibility: hidden;
  white-space: nowrap;
  font-family: 'Libre Caslon Text', serif;
  font-size: 1em;
  font-weight: 400;
  text-transform: uppercase;
}


</style>