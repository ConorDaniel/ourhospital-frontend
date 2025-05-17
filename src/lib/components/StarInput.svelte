<script lang="ts">
    export let score: number;
    export let onChange: (val: number) => void;
  </script>
  
  <div class="stars">
    {#each [1, 2, 3, 4, 5] as value}
      <span
        class="star {score >= value ? 'active' : ''}"
        on:click={() => onChange(value)}
      >★</span>
    {/each}
  </div>
  
  <style>
    .stars {
      font-size: 1.5rem;
      color: #ccc;
      cursor: pointer;
    }
    .star.active {
      color: gold;
    }
  </style>
  