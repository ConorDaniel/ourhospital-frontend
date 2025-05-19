<script lang="ts">
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
  
    // Bind file inputs
    let image1Input: HTMLInputElement;
    let image2Input: HTMLInputElement;
    let image3Input: HTMLInputElement;
  
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
            Authorization: `Bearer ${localStorage.getItem("jwt")}`
          },
          body: formData
        });
  
        if (response.ok) {
          message = "✅ Hospital added successfully!";
        } else {
          message = "❌ Submission failed: " + (await response.text());
        }
      } catch (err) {
        message = "❌ Network error: " + err.message;
      }
    }
  </script>
  
  <section class="section">
    <h1 class="title">Admin: Add Hospital to Master List</h1>
  
    {#if message}
      <p class="notification is-info">{message}</p>
    {/if}
  
    <form on:submit={handleSubmit} enctype="multipart/form-data">
      <div class="field">
        <label class="label" for="hospital-name">Hospital Name</label>
        <input id="hospital-name" class="input" bind:value={name} required />
      </div>
  
      <div class="field">
        <label class="label" for="hospital-type">Type</label>
        <div class="control">
          <div class="select">
            <select id="hospital-type" bind:value={type} required>
              <option value="National">National</option>
              <option value="Regional">Regional</option>
              <option value="Local">Local</option>
              <option value="Other">Other</option>
            </select>
          </div>
        </div>
      </div>
  
      <div class="field">
        <label class="label" for="hospital-location">Location</label>
        <input id="hospital-location" class="input" bind:value={location} required />
      </div>
  
      <div class="field">
        <label class="label" for="latitude">Latitude</label>
        <input id="latitude" class="input" type="number" bind:value={latitude} required step="any" />
      </div>
  
      <div class="field">
        <label class="label" for="longitude">Longitude</label>
        <input id="longitude" class="input" type="number" bind:value={longitude} required step="any" />
      </div>
  
      <div class="field">
        <label class="label" for="staffCount">Staff Count</label>
        <input id="staffCount" class="input" type="number" bind:value={staffCount} required />
      </div>
  
      <div class="field">
        <label class="label" for="budget">Budget (€M)</label>
        <input id="budget" class="input" type="number" bind:value={budget} required step="0.01" />
      </div>
  
      <div class="field">
        <label class="label" for="bedCount">Bed Count</label>
        <input id="bedCount" class="input" type="number" bind:value={bedCount} required />
      </div>
  
      <div class="field">
        <label class="label" for="region">Region (0–6)</label>
        <input id="region" class="input" type="number" bind:value={region} required min="0" max="6" />
      </div>
  
      <div class="field">
        <label class="label" for="image1">Select Image 1</label>
        <input id="image1" type="file" accept="image/*" bind:this={image1Input} />
      </div>
  
      <div class="field">
        <label class="label" for="image2">Select Image 2</label>
        <input id="image2" type="file" accept="image/*" bind:this={image2Input} />
      </div>
  
      <div class="field">
        <label class="label" for="image3">Select Image 3</label>
        <input id="image3" type="file" accept="image/*" bind:this={image3Input} />
      </div>
  
      <div class="field">
        <button class="button is-primary" type="submit">Add Hospital</button>
      </div>
    </form>
  </section>
  