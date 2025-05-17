<script lang="ts">
	import { onMount } from "svelte";
	import HospitalMap from "$lib/components/HospitalMap.svelte";
	import ImageRotator from "$lib/components/ImageRotator.svelte";
  
	let hospitals = [];
	let error = "";
	let searchTerm = "";
  
	// Load all hospitals (no login required)
	onMount(async () => {
	  try {
		const res = await fetch("http://localhost:3000/api/hospitals");
		if (res.ok) {
		  hospitals = await res.json();
		} else {
		  error = "Failed to load hospitals: " + (await res.text());
		}
	  } catch (err) {
		error = "Network error: " + err.message;
	  }
	});
  
	// Filter hospitals by search term
	$: filteredHospitals = hospitals.filter(h =>
	  h.name.toLowerCase().includes(searchTerm.trim().toLowerCase())
	);
  
	// Show alert on Departments click
	function handleDeptClick() {
	  alert("Please log in or sign up to access department details.");
	}
  </script>
  
  <nav class="navbar" role="navigation" aria-label="main navigation">
	<div class="navbar-brand">
	  <img src="/images/logo.jpg" alt="Hospital logo" style="max-height: 64px; margin: 10px;" />
	  <span class="title is-3 has-text-primary" style="margin-left: 10px;">OurHospitals</span>
	</div>
  
	<div class="navbar-menu">
	  <div class="navbar-end">
		<div class="navbar-item">
		  <div class="buttons">
			<a class="button" href="/login">Log in</a>
			<a class="button" href="/signup">Sign up</a>
		  </div>
		</div>
	  </div>
	</div>
  </nav>
  
  <section class="section has-text-centered">
	<h1 class="title is-3">Who works where in Irish Hospitals</h1>
	<h1 class="title is-3">Sign up or Log in...</h1>
	<p class="subtitle is-6 has-text-grey">Sign-up requires admin approval.</p>
  
	<figure class="image is-inline-block" style="max-width: 400px; margin-top: 20px;">
	  <img src="/images/chi.jpg" alt="Hospital image" class="is-rounded" />
	</figure>
  </section>
  
  <section class="section">
	<h2 class="title is-4">Browse Hospitals</h2>
  
	<div class="field mb-5" style="max-width: 400px;">
	  <label class="label">Search hospitals by name</label>
	  <div class="control">
		<input
		  class="input"
		  type="text"
		  placeholder="e.g. Mater"
		  bind:value={searchTerm}
		/>
	  </div>
	</div>
  
	{#if error}
	  <p class="notification is-danger">{error}</p>
	{:else if filteredHospitals.length === 0}
	  <p class="notification is-warning">No matching hospitals found.</p>
	{:else}
	{#each filteredHospitals as hospital}
  <div class="box mb-5">
    <!-- Row 1: Name and Button -->
    <div class="columns is-vcentered is-mobile is-multiline mb-2">
      <div class="column is-8">
        <h2 class="title is-4 has-text-weight-bold is-uppercase">
          {hospital.name}
        </h2>
      </div>
      <div class="column is-4 has-text-right">
        <button class="button is-info" on:click={handleDeptClick}>
          Departments
        </button>
      </div>
    </div>

    <!-- Row 2: Image, Rate Button, Map -->
    <div class="columns is-variable is-1 is-multiline">
      <div class="column is-one-third">
        <ImageRotator
          images={[
            "/images/hospitals/mater-1a.jpg",
            "/images/hospitals/mater-1b.jpg",
            "/images/hospitals/mater-1c.jpg"
          ]}
        />

        <!-- 📌 Rate button under images -->
        <div class="has-text-centered mt-3">
          <a
            class="button is-link is-light"
            href={`/rate?hospitalId=${hospital._id}`}
          >
            ⭐ Rate this hospital
          </a>
        </div>
      </div>

      <div class="column is-two-thirds">
        <div class="map-wrapper">
          {#key hospital._id}
            <HospitalMap lat={hospital.latitude} lng={hospital.longitude} />
          {/key}
        </div>
      </div>
    </div>
  </div>
{/each}
	{/if}
  </section>
  
  <style>
	.map-wrapper {
	  height: 300px;
	  position: relative;
	  border-radius: 0.5rem;
	  border: 1px solid #ddd;
	  overflow: hidden;
	}
  </style>
  