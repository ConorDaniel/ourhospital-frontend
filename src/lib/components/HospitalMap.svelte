<script lang="ts">
  import { onMount } from "svelte";
  import { tick } from "svelte";

  export let lat: number;
  export let lng: number;

  let mapDiv: HTMLDivElement;
  let map: any;

  function initializeMap(L) {
    map = L.map(mapDiv).setView([lat, lng], 15);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "&copy; OpenStreetMap contributors",
      maxZoom: 18,
    }).addTo(map);

    L.marker([lat, lng])
      .addTo(map)
      .bindPopup("Hospital Location")
      .openPopup();
  }

  async function ensureRendered() {
    await tick();
    const L = await import("leaflet");

    const ro = new ResizeObserver(() => {
      if (map) {
        map.invalidateSize();
      }
    });
    ro.observe(mapDiv);

    initializeMap(L);
    setTimeout(() => map.invalidateSize(), 300); // safety re-check
  }

  onMount(() => {
    if (typeof window !== "undefined") {
      ensureRendered();
    }
  });
</script>

<style>
  .map-container {
    height: 300px;
    width: 100%;
    border-radius: 0.5rem;
    border: 1px solid #ddd;
    overflow: hidden;
  }
</style>

<div bind:this={mapDiv} class="map-container"></div>
