<script>
	import * as topojson from 'topojson-client';
	import { geoPath } from 'd3-geo';
  import { geoConicConformalSpain } from 'd3-composite-projections';
	import { draw } from 'svelte/transition';
  import municipalities from "$data/municipalities.json";
  import aeropuertos from "$data/aeropuertos.js";
  import { scaleLinear } from 'd3-scale';
  import { max } from 'd3-array';
  import Tooltip from '$components/Tooltip.svelte';

  let provincias = topojson.feature(municipalities, municipalities.objects.provinces).features;
  let bordes = topojson.mesh(municipalities, municipalities.objects.autonomous_regions, (a, b) => a !== b);

  $: compositionBorders = projection.getCompositionBorders();

  let width = 400;
  $: height = width * 0.70;
  
  $: projection = geoConicConformalSpain().scale(width > 1200 ? 4000 : width * 3.5).translate(width > 1200 ? [width / 2, 400] : [width / 2, height / 2]);
  $: path = geoPath(projection);

  const sizeScale = scaleLinear()
    .domain([0, max(aeropuertos, d => d.Total)])
    .range([2.5, 40]);

  const colorScale = scaleLinear()
    .domain([0, max(aeropuertos, d => d.Total)])
    .range(["#FFBE00", "#FE7F09"])

  const colorBorderScale = scaleLinear()
    .domain([0, max(aeropuertos, d => d.Total)])
    .range(["#BF8E00", "#B35400"])

  let selection;

  window.addEventListener("DOMContentLoaded", (event) => {
	function updateIframeHeight() {
		const el = document.documentElement;
		const rect = el.getBoundingClientRect();
		const styles = window.getComputedStyle(el);
		const margin = parseFloat(styles.marginTop) + parseFloat(styles.marginBottom);
		const height = Math.ceil(rect.height + margin);

		window.parent.postMessage({
			type: "resize-iframe",
			value: height
		}, "*");
	}
	updateIframeHeight();

	if (window.ResizeObserver) {
		new ResizeObserver(() => {
			updateIframeHeight();
		}).observe(document.documentElement);
	} else {
		window.addEventListener('load', updateIframeHeight);
		window.addEventListener('resize', updateIframeHeight);
	}

	window.addEventListener("message", (event) => {
		if (event.data.type === "request-resize") {
			updateIframeHeight();
		}
	}, false);
});

</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class="chart-container" bind:clientWidth={width}
on:click={() => {
  selection = null;
}}>
  <svg {width} {height} on:click|stopPropagation>
    {#each provincias as provincia}
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <path d={path(provincia)} fill="#EBEBEB" stroke="black" stroke-width="0.05"/>
    {/each}

    <path d={path(bordes)} fill="none" stroke="black" stroke-width="0.1" />
    <path d={compositionBorders} fill="none" stroke="black" stroke-width="0.5"/>

    {#each aeropuertos as aeropuerto}
    <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
    <!-- svelte-ignore a11y-mouse-events-have-key-events -->
    <circle
    cx={projection([aeropuerto.Longitude, aeropuerto.Latitude])[0]}
    cy={projection([aeropuerto.Longitude, aeropuerto.Latitude])[1]}
    r={sizeScale(aeropuerto.Total)}
    fill={colorScale(aeropuerto.Total)}
    fill-opacity={ selection ? selection === aeropuerto ? "100%" : "30%" : "70%"}
    stroke={colorBorderScale(aeropuerto.Total)}
    stroke-width="0.8"
    stroke-opacity={ selection ? selection === aeropuerto ? "100%" : "70%" : "100%"}
    on:mouseover={() => {
      selection = aeropuerto;
    }}
    on:focus={() => {
      selection = aeropuerto;
    }}
    tabindex="0"
    on:mouseout={() => {
      selection = null;
    }}
    />
    {/each}
  </svg>

  {#if selection}
    <Tooltip {projection} {selection} {width} {height}/>
  {/if}

</div>


<style>
  .chart-container {
    max-height: 810px;
  }

  svg {
    max-height: 810px;
  }

</style>