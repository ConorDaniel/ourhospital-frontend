<script lang="ts">
    import { onMount } from "svelte";
    export let images: string[] = [];
  
    let current = 0;
    let interval;
  
    onMount(() => {
      if (images.length > 1) {
        interval = setInterval(() => {
          current = (current + 1) % images.length;
        }, 3000);
      }
  
      return () => clearInterval(interval);
    });
  </script>
  
  {#if images.length > 0}
    <figure class="image is-128x128">
      <img src={images[current]} alt="Rotating image" />
    </figure>
  {:else}
    <p>No images available</p>
  {/if}
  