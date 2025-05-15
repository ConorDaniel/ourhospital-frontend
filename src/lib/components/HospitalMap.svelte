<script lang="ts">
  import { onMount } from "svelte";
  export let lat: number;
  export let lng: number;

  let mapDiv: HTMLDivElement;

  onMount(async () => {
    // ✅ SSR-safe dynamic import
    const L = await import("leaflet");

    const map = L.map(mapDiv).setView([lat, lng], 14);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: '&copy; OpenStreetMap contributors'
    }).addTo(map);

    L.marker([lat, lng]).addTo(map);
  });
</script>

<!-- ✅ Styling for embedded display -->
<div
  bind:this={mapDiv}
  class="leaflet-container"
  style="
    height: 250px;
    width: 100%;
    border-radius: 0.5rem;
    overflow: hidden;
    z-index: 1;
  "
></div>
