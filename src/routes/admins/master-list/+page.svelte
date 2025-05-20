<script lang="ts">
  import { onMount } from "svelte";
  import { goto } from "$app/navigation";

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

  let hospitals = [];
  let userEmail = "";
  let pictureUrl = "";
  let error = "";
  let token = "";

  let image1Input: HTMLInputElement;
  let image2Input: HTMLInputElement;
  let image3Input: HTMLInputElement;

  onMount(async () => {
    token = localStorage.getItem("token") || "";
    if (!token) {
      error = "Not logged in.";
      return;
    }

    try {
      const userRes = await fetch("http://localhost:3000/api/users/me", {
        headers: { Authorization: `Bearer ${token}` },
      });

      if (userRes.ok) {
        const user = await userRes.json();
        userEmail = user.email;
        pictureUrl = user.pictureUrl?.trim() ||
          "https://res.cloudinary.com/dycaquyie/image/upload/v1747570490/Screenshot_2025-05-18_at_13.13.48_dywns0.png";
      }

      const hospitalRes = await fetch("http://localhost:3000/api/hospitals", {
        headers: { Authorization: `Bearer ${token}` },
      });

      if (hospitalRes.ok) {
        hospitals = await hospitalRes.json();
      } else {
        error = "Failed to load hospitals.";
      }
    } catch (err) {
      error = "Network error: " + err.message;
    }
  });

  function handleLogout() {
    localStorage.removeItem("token");
    goto("/");
  }

  function goToDashboard() {
    goto("/dashboard");
  }

  async function handleSubmit(event: Event) {
    event.preventDefault();
    message = "";

    const formData = new FormData();
    formData.append("name", name);
    formData.append("type", type);
    formData.append("location", location);
    formData.append("latitude", String(latitude));
    formData.append("longitude", String(longitude));
    formData.append("staffCount", String(staffCount));
    formData.append("budget", String(budget));
    formData.append("bedCount", String(bedCount));
    formData.append("region", String(region));

    if (image1Input?.files?.length) formData.append("image1", image1Input.files[0]);
    if (image2Input?.files?.length) formData.append("image2", image2Input.files[0]);
    if (image3Input?.files?.length) formData.append("image3", image3Input.files[0]);

    try {
      const response = await fetch("http://localhost:3000/api/hospitals", {
        method: "POST",
        headers: {
          Authorization: `Bearer ${token}`
        },
        body: formData
      });

      if (response.ok) {
        message = "✅ Hospital added successfully!";
        // Refresh list
        const updated = await fetch("http://localhost:3000/api/hospitals", {
          headers: { Authorization: `Bearer ${token}` },
        });
        hospitals = await updated.json();
      } else {
        message = "❌ Submission failed: " + (await response.text());
      }
    } catch (err) {
      message = "❌ Network error: " + err.message;
    }
  }
</script>

<style>
  .user-info {
    display: flex;
    align-items: center;
    margin-left: 2rem;
  }

  .user-info img {
    height: 40px;
    width: 40px;
    border-radius: 50%;
    margin-right: 0.75rem;
  }
</style>

<section class="section">
  <!-- Header -->
  <nav class="level mb-5">
    <div class="level-left">
      <div class="level-item">
        <img src="/images/logo.png" alt="Logo" style="height: 50px; margin-right: 10px;" />
        <span class="title is-4 has-text-primary">ourHospital Admin Panel</span>
      </div>
      {#if userEmail}
        <div class="level-item user-info">
          <img src={pictureUrl} alt="User Picture" />
          <span class="is-size-6">Logged in as <strong>{userEmail}</strong></span>
        </div>
      {/if}
    </div>
    <div class="level-right">
      <div class="buttons">
        <button class="button is-light" on:click={goToDashboard}>🏠 Dashboard</button>
        <button class="button is-danger" on:click={handleLogout}>🚪 Logout</button>
      </div>
    </div>
  </nav>

  <!-- Title -->
  <h2 class="title is-3">Add Hospital to Master List</h2>

  {#if message}
    <p class="notification is-info">{message}</p>
  {/if}
  {#if error}
    <p class="notification is-danger">{error}</p>
  {/if}

  <!-- Hospital Form -->
  <form on:submit={handleSubmit} enctype="multipart/form-data">
    <div class="field">
      <label class="label">Hospital Name</label>
      <input class="input" bind:value={name} required />
    </div>

    <div class="field">
      <label class="label">Type</label>
      <div class="select">
        <select bind:value={type} required>
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
      <input class="input" type="number" bind:value={region} required min="0" max="6" />
    </div>

    <div class="field">
      <label class="label">Image 1</label>
      <input type="file" accept="image/*" bind:this={image1Input} />
    </div>

    <div class="field">
      <label class="label">Image 2</label>
      <input type="file" accept="image/*" bind:this={image2Input} />
    </div>

    <div class="field">
      <label class="label">Image 3</label>
      <input type="file" accept="image/*" bind:this={image3Input} />
    </div>

    <div class="field">
      <button class="button is-primary" type="submit">Add Hospital</button>
    </div>
  </form>

  <hr class="my-5" />

  <h2 class="title is-4">Existing Hospitals</h2>
  {#if hospitals.length === 0}
    <p class="notification is-warning">No hospitals found.</p>
  {:else}
    <ul>
      {#each hospitals as hospital}
        <li><strong>{hospital.name}</strong> — {hospital.location} ({hospital.type})</li>
      {/each}
    </ul>
  {/if}
</section>