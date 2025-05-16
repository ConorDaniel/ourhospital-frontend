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

  const deleteHospital = async (id: string) => {
    const token = localStorage.getItem("jwt");
    if (!token) {
      error = "Not logged in.";
      return;
    }

    const confirmDelete = confirm("Are you sure you want to delete this hospital?");
    if (!confirmDelete) return;

    try {
      const res = await fetch(`http://localhost:3000/api/hospitals/${id}`, {
        method: "DELETE",
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });

      if (res.ok) {
        hospitals = hospitals.filter((h) => h._id !== id);
      } else {
        error = `Delete failed: ${await res.text()}`;
      }
    } catch (err) {
      error = `Network error: ${err.message}`;
    }
  };

  const randomImage = () => {
    const images = ["1a.jpg", "1b.jpg", "1c.jpg"];
    return `/images/hospitals/${images[Math.floor(Math.random() * images.length)]}`;
  };
</script>

<style>
  .map-wrapper {
    height: 300px;
    position: relative;
    border-radius: 0.5rem;
    border: 1px solid #ddd;
    overflow: hidden;
  }
</style>

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
      <div class="box mb-5">
        <!-- Row 1: Name and Buttons -->
        <div class="columns is-vcentered is-mobile is-multiline mb-2">
          <div class="column is-8">
            <h2 class="title is-4 has-text-weight-bold is-uppercase">
              {hospital.name}
            </h2>
          </div>
          <div class="column is-4 has-text-right">
            <a href={`/departments?hospitalId=${hospital._id}`} class="button is-info">
              Departments
            </a>            
            <button
              class="button is-danger"
              disabled={hospital.departments && hospital.departments.length > 0}
              on:click={() => deleteHospital(hospital._id)}
            >
              Delete
            </button>
          </div>
        </div>

        <!-- Row 2: Image + Map -->
        <div class="columns is-variable is-1 is-multiline">
          <div class="column is-one-third">
            <ImageRotator
              images={[
                "/images/hospitals/mater-1a.jpg",
                "/images/hospitals/mater-1b.jpg",
                "/images/hospitals/mater-1c.jpg"
              ]}
            />
          </div>
          <div class="column is-two-thirds">
            <div class="map-wrapper">
              <HospitalMap lat={hospital.latitude} lng={hospital.longitude} />
            </div>
          </div>
        </div>
      </div>
    {/each}
  {/if}
</section>
