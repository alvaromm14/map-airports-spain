<script>
    export let selection;
    export let width;
    export let projection;
    export let height;

    import { fly, fade } from "svelte/transition";

    let tooltipWidth;

    const xNudge = 5;
    const yNudge = 5;

    $: selectionX = projection([selection.Longitude, selection.Latitude])[0];
    $: selectionY = projection([selection.Longitude, selection.Latitude])[1];

    $: xPosition = selectionX + tooltipWidth + xNudge > width
        ? Math.max(selectionX - tooltipWidth - xNudge, xNudge)
        : selectionX + xNudge;

        $: yPosition = selectionY + 80 > height ? selectionY - 70 : selectionY + yNudge;

</script>

<div
    class="tooltip"
    in:fly={{ y: 10, duration: 120, delay: 120 }}
    out:fade
    style="position: absolute;
        top: {yPosition}px;
        left: {xPosition}px;"
    bind:clientWidth={tooltipWidth}
>
    <h1>{selection.AEROPUERTOS}</h1><h2>{(Math.round(selection.Total / 100) * 100).toLocaleString('es-ES')} pasajeros</h2>
    <div class="info">
        <span class="cambio" style="background:#F89F4C; color:white; font-weight:800">Cambio desde 2019: {(selection.Cambio2019 * 100).toFixed(0)}%</span>
    </div>

</div>

<style>
    .tooltip {
        background-color: white;
        box-shadow: 2px 3px 8px rgba(0, 0, 0, 0.15);
        padding: 8px 6px;
        border-radius: 3px;
        pointer-events: none;
        transition: top 50ms ease, left 50ms ease;
    }

    .info {
        display: flex;
        justify-content: space-between;
        column-gap: 8px;
        align-items: center;
    }

    .cambio {
        font-size: 11px;
        padding: 3px;
        border-radius: 3px;
        text-transform: uppercase;
    }

    h1 {
        font-size: 1rem;
        font-weight: 500;
        margin-bottom: 4px;
    }

    h2 {
        font-size: 0.75rem;
        font-weight: 400;
        font-style: italic;
        margin-bottom: 6px;
    }
</style>