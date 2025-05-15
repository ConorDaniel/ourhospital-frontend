<script lang="ts">
    import { goto } from "$app/navigation";
    import { onMount } from "svelte";
  
    let name = "";
    let latitude: number | null = null;
    let longitude: number | null = null;
    let type = "";
    let imagePrefix = "";
    let error = "";
  
    async function handleSubmit(event: Event) {
      event.preventDefault();
      error = "";
  
      if (!name || !latitude || !longitude || !type || type === "select") {
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
        latitude,
        longitude,
        type,
        images: [
          `/images/hospitals/${imagePrefix}-1a.jpg`,
          `/images/hospitals/${imagePrefix}-1b.jpg`,
          `/images/hospitals/${imagePrefix}-1c.jpg`
        ]
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
    <!-- Header -->
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
        <label class="label">Name</label>
        <div class="control">
          <input class="input" type="text" bind:value={name} required />
        </div>
      </div>
  
      <div class="columns">
        <div class="column">
          <label class="label">Latitude</label>
          <input class="input" type="number" bind:value={latitude} required step="any" />
        </div>
        <div class="column">
          <label class="label">Longitude</label>
          <input class="input" type="number" bind:value={longitude} required step="any" />
        </div>
      </div>
  
      <div class="field">
        <label class="label">Hospital Type</label>
        <div class="control">
          <div class="select">
            <select bind:value={type} required>
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
        <label class="label">Image Prefix</label>
        <div class="control">
          <input class="input" type="text" bind:value={imagePrefix} placeholder="e.g. mater" required />
        </div>
        <p class="help">This will generate 3 images: {imagePrefix}-1a.jpg, -1b.jpg, -1c.jpg</p>
      </div>
  
      <div class="field is-grouped">
        <div class="control">
          <button class="button is-primary" type="submit">Submit</button>
        </div>
      </div>
    </form>
  </section>
  