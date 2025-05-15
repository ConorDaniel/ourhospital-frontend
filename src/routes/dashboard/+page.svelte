<script lang="ts">
  import { onMount } from "svelte";
  import HospitalMap from "$lib/components/HospitalMap.svelte";
  import ImageRotator from "$lib/components/ImageRotator.svelte";

  let hospitals = [];
  let error = "";

  onMount(async () => {
    const token = localStorage.getItem("jwt");
    if (!token) {
      error = "Not logged in.";
      return;
    }

    try {
      const res = await fetch("http://localhost:3000/api/my-hospitals", {
        headers: { Authorization: `Bearer ${token}` },
      });
      if (res.ok) {
        hospitals = await res.json();
      } else {
        error = "Failed to load hospitals: " + (await res.text());
      }
    } catch (err) {
      error = "Network error: " + err.message;
    }
  });

  const randomImage = () => {
    const images = ["1a.jpg", "1b.jpg", "1c.jpg"];
    return `/images/hospitals/${images[Math.floor(Math.random() * images.length)]}`;
  };
</script>

<section class="section">
  <!-- Header Row -->
  <nav class="level mb-5">
    <div class="level-left">
      <div class="level-item">
        <img src="/images/logo.jpg" alt="Logo" style="height: 50px; margin-right: 10px" />
        <span class="title is-4 has-text-primary">ourHospital</span>
      </div>
    </div>
    <div class="level-right">
      <div class="buttons">
        <a href="/about" class="button is-light">About</a>
        <a href="/add-hospital" class="button is-link">Add Hospital</a>
        <a href="/login" class="button is-danger">Logout</a>
      </div>
    </div>
  </nav>

  <h1 class="title">Hospital Dashboard</h1>

  {#if error}
    <p class="notification is-danger">{error}</p>
  {:else if hospitals.length === 0}
    <p class="notification is-warning">No hospitals found for this user.</p>
  {:else}
    {#each hospitals as hospital}
      <div class="box">
        <h2 class="title is-5">{hospital.name}</h2>
        <div class="columns is-vcentered">
          <!-- Image -->
          <div class="column is-one-third">
            <figure class="image">
              <img src={randomImage()} alt="Hospital photo" />
            </figure>
          </div>

          <!-- Map -->
          <div class="column is-one-third">
            <HospitalMap lat={hospital.latitude} lng={hospital.longitude} />
          </div>

          <!-- Department Button -->
          <div class="column is-one-third has-text-centered">
            <a href="/departments/{hospital._id}" class="button is-info is-medium">Departments</a>
          </div>
        </div>
      </div>
    {/each}
  {/if}
</section>
