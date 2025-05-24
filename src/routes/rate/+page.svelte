<script lang="ts">
  import { onMount } from "svelte";
  import { page } from "$app/stores";
  import { goto } from "$app/navigation";
  import StarInput from "$lib/components/StarInput.svelte";

  let hospitalId = "";
  let hospitalName = "";
  let care = 3;
  let cleanliness = 3;
  let friendliness = 3;
  let food = 3;
  let message = "";

  $: hospitalId = $page.url.searchParams.get("hospitalId") || "";

  onMount(async () => {
    if (!hospitalId) return;
    const res = await fetch(`${import.meta.env.VITE_API_URL}/api/hospitals/${hospitalId}`);
    if (res.ok) {
      const hospital = await res.json();
      hospitalName = hospital.name;
    } else {
      hospitalName = "Unknown Hospital";
    }
  });

  const submitRating = async () => {
    const res = await fetch(`${import.meta.env.VITE_API_URL}/api/hospitals/${hospitalId}/ratings`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ care, cleanliness, friendliness, food })
    });

    if (res.ok) {
      message = "Thanks for your feedback!";
      setTimeout(() => {
        goto(`/?highlight=${hospitalId}`);  // 👈 send hospitalId to home
      }, 2000);
    } else {
      message = "Something went wrong submitting your rating.";
    }
  };
</script>

<section class="section">
  <h1 class="title">Rate this hospital</h1>
  <p class="subtitle is-5 has-text-weight-semibold mb-5">{hospitalName}</p>

  <div class="field">
    <label class="label">Care</label>
    <StarInput score={care} onChange={(val) => care = val} />
    <span>{care} / 5</span>
  </div>

  <div class="field">
    <label class="label">Cleanliness</label>
    <StarInput score={cleanliness} onChange={(val) => cleanliness = val} />
    <span>{cleanliness} / 5</span>
  </div>

  <div class="field">
    <label class="label">Friendliness</label>
    <StarInput score={friendliness} onChange={(val) => friendliness = val} />
    <span>{friendliness} / 5</span>
  </div>

  <div class="field">
    <label class="label">Food</label>
    <StarInput score={food} onChange={(val) => food = val} />
    <span>{food} / 5</span>
  </div>

  <div class="field mt-4">
    <button class="button is-primary" on:click={submitRating}>
      Submit Rating
    </button>
  </div>

  {#if message}
    <p class="notification is-info mt-4">{message}</p>
  {/if}
</section>
