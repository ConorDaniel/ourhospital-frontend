<script lang="ts">
  import { goto } from "$app/navigation";

  let name = "";
  let type = "";
  let location = "";
  let latitude: number | null = null;
  let longitude: number | null = null;
  let error = "";

  async function handleSubmit(event: Event) {
    event.preventDefault();
    error = "";

    if (!name || !type || !location || latitude === null || longitude === null || type === "select") {
      error = "Please complete all fields.";
      return;
    }

    const token = localStorage.getItem("jwt");
    if (!token) {
      error = "Authentication required.";
      return;
    }

    const newHospital = {
      name,
      type,
      location,
      latitude,
      longitude
    };

    try {
      const res = await fetch("http://localhost:3000/api/hospitals", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${token}`
        },
        body: JSON.stringify(newHospital)
      });

      if (res.ok) {
        goto("/dashboard");
      } else {
        const result = await res.json();
        error = result.message || "Failed to add hospital.";
      }
    } catch (err) {
      error = "Network error: " + err.message;
    }
  }
</script>

<section class="section">
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
        <a href="/dashboard" class="button is-link">Dashboard</a>
        <a href="/login" class="button is-danger">Logout</a>
      </div>
    </div>
  </nav>

  <h1 class="title">Add Hospital</h1>

  {#if error}
    <p class="notification is-danger">{error}</p>
  {/if}

  <form on:submit={handleSubmit}>
    <div class="field">
      <label class="label" for="name">Name</label>
      <div class="control">
        <input class="input" id="name" type="text" bind:value={name} required />
      </div>
    </div>

    <div class="field">
      <label class="label" for="type">Hospital Type</label>
      <div class="control">
        <div class="select">
          <select id="type" bind:value={type} required>
            <option value="select">Select</option>
            <option value="National">National</option>
            <option value="Regional">Regional</option>
            <option value="Local">Local</option>
            <option value="Other">Other</option>
          </select>
        </div>
      </div>
    </div>

    <div class="field">
      <label class="label" for="location">Location</label>
      <div class="control">
        <input class="input" id="location" type="text" bind:value={location} required />
      </div>
    </div>

    <div class="columns">
      <div class="column">
        <label class="label" for="latitude">Latitude</label>
        <input class="input" id="latitude" type="number" bind:value={latitude} required step="any" />
      </div>
      <div class="column">
        <label class="label" for="longitude">Longitude</label>
        <input class="input" id="longitude" type="number" bind:value={longitude} required step="any" />
      </div>
    </div>

    <div class="field is-grouped">
      <div class="control">
        <button class="button is-primary" type="submit">Submit</button>
      </div>
    </div>
  </form>
</section>
