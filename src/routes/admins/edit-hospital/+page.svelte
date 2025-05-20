<script lang="ts">
  import { onMount } from "svelte";
  import { page } from "$app/stores";
  import { goto } from "$app/navigation";
  import { get } from "svelte/store";

  let hospitalId = get(page).url.searchParams.get("id");
  let hospital: any = {};

  let name = "";
  let type = "Local";
  let location = "";
  let latitude = 0;
  let longitude = 0;
  let staffCount = 0;
  let budget = 0;
  let bedCount = 0;
  let region = 6;

  let message = "";
  let error = "";

  onMount(async () => {
    const token = localStorage.getItem("token");
    if (!token) {
      error = "Not logged in.";
      return;
    }

    try {
      // ✅ Verify user role
      const userRes = await fetch("http://localhost:3000/api/users/me", {
        headers: { Authorization: `Bearer ${token}` },
      });

      if (!userRes.ok) {
        error = "Failed to verify user.";
        return;
      }

      const user = await userRes.json();
      if (user.role !== "admin") {
        error = "Access denied: Admins only.";
        return;
      }

      // ✅ Load hospital details
      const res = await fetch(`http://localhost:3000/api/hospitals/${hospitalId}`);
      if (res.ok) {
        hospital = await res.json();
        ({ name, type, location, latitude, longitude, staffCount, budget, bedCount, region } = hospital);
      } else {
        error = "Failed to load hospital details.";
      }
    } catch (err) {
      error = "Network error: " + err.message;
    }
  });

  async function handleUpdate(event: Event) {
    event.preventDefault();
    message = "";
    error = "";

    const token = localStorage.getItem("token");
    if (!token) {
      error = "Not logged in.";
      return;
    }

    const payload = {
      name,
      type,
      location,
      latitude,
      longitude,
      staffCount,
      budget,
      bedCount,
      region
    };

    try {
      const res = await fetch(`http://localhost:3000/api/hospitals/${hospitalId}`, {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${token}`
        },
        body: JSON.stringify(payload)
      });

      if (res.ok) {
        message = "✅ Hospital updated successfully!";
        setTimeout(() => goto("/admins/master-list"), 1000);
      } else {
        const text = await res.text();
        error = "❌ Update failed: " + text;
      }
    } catch (err) {
      error = "❌ Network error: " + err.message;
    }
  }
</script>

<section class="section">
  <h1 class="title">Edit Hospital</h1>

  {#if error}
    <p class="notification is-danger">{error}</p>
  {/if}
  {#if message}
    <p class="notification is-success">{message}</p>
  {/if}

  <form on:submit={handleUpdate}>
    <div class="field">
      <label class="label">Name</label>
      <input class="input" bind:value={name} required />
    </div>

    <div class="field">
      <label class="label">Type</label>
      <div class="select">
        <select bind:value={type}>
          <option value="National">National</option>
          <option value="Regional">Regional</option>
          <option value="Local">Local</option>
          <option value="Other">Other</option>
        </select>
      </div>
    </div>

    <div class="field">
      <label class="label">Location</label>
      <input class="input" bind:value={location} required />
    </div>

    <div class="field">
      <label class="label">Latitude</label>
      <input class="input" type="number" step="any" bind:value={latitude} required />
    </div>

    <div class="field">
      <label class="label">Longitude</label>
      <input class="input" type="number" step="any" bind:value={longitude} required />
    </div>

    <div class="field">
      <label class="label">Staff Count</label>
      <input class="input" type="number" bind:value={staffCount} required />
    </div>

    <div class="field">
      <label class="label">Budget (€M)</label>
      <input class="input" type="number" step="0.01" bind:value={budget} required />
    </div>

    <div class="field">
      <label class="label">Bed Count</label>
      <input class="input" type="number" bind:value={bedCount} required />
    </div>

    <div class="field">
      <label class="label">Region (0–6)</label>
      <input class="input" type="number" min="0" max="6" bind:value={region} required />
    </div>

    <div class="field mt-4">
      <button class="button is-primary" type="submit">Save Changes</button>
      <a href="/admins/master-list" class="button is-light ml-3">Cancel</a>
    </div>
  </form>
</section>
