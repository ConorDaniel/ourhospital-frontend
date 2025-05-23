<script lang="ts">
  import { onMount } from "svelte";

  let firstName = "";
  let lastName = "";
  let email = "";
  let password = "";
  let message = "";
  let pictureInput: HTMLInputElement;

  let hospitals = [];
  let selectedHospitals: string[] = [];

  onMount(async () => {
    try {
      const response = await fetch(`${import.meta.env.VITE_API_URL}/api/hospitals`);
      if (response.ok) {
        hospitals = await response.json();
      } else {
        message = "❌ Failed to load hospital list";
      }
    } catch (err) {
      message = "❌ Network error: " + err.message;
    }
  });

  function toggleHospital(id: string) {
    if (selectedHospitals.includes(id)) {
      selectedHospitals = selectedHospitals.filter(h => h !== id);
    } else {
      selectedHospitals = [...selectedHospitals, id];
    }
  }

  async function handleSignup(event: Event) {
    event.preventDefault();
    message = "";

    if (selectedHospitals.length === 0) {
      message = "❌ Please select at least one hospital.";
      return;
    }

    let pictureUrl = "";

    if (pictureInput?.files?.length) {
      const file = pictureInput.files[0];
      const formData = new FormData();
      formData.append("file", file);

      try {
        const uploadRes = await fetch(`${import.meta.env.VITE_API_URL}/api/images/upload`, {
          method: "POST",
          body: formData,
          headers: {
            Authorization: `Bearer ${localStorage.getItem("jwt") || ""}`
          }
        });

        if (uploadRes.ok) {
          const result = await uploadRes.json();
          pictureUrl = result.secure_url;
        } else {
          const error = await uploadRes.text();
          message = `❌ Image upload failed: ${error}`;
          return;
        }
      } catch (err) {
        message = "❌ Network error during image upload: " + err.message;
        return;
      }
    }

    try {
      const response = await fetch(`${import.meta.env.VITE_API_URL}/api/users`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          firstName,
          lastName,
          email,
          password,
          role: "user",
          hospitals: selectedHospitals,
          pictureUrl
        })
      });

      if (response.ok) {
        message = "✅ Sign-up successful! Redirecting...";
        setTimeout(() => {
          window.location.href = "/login";
        }, 1500);
      } else {
        const error = await response.text();
        message = "❌ Sign-up failed: " + error;
      }
    } catch (err) {
      message = "❌ Network error: " + err.message;
    }
  }
</script>

<nav class="navbar" role="navigation" aria-label="main navigation">
  <div class="navbar-brand">
    <img src="/images/logo.jpg" alt="Hospital logo" style="max-height: 64px; margin: 10px;" />
    <span class="title is-3 has-text-primary" style="margin-left: 10px;">ourHospital</span>
  </div>

  <div class="navbar-menu">
    <div class="navbar-end">
      <div class="navbar-item">
        <div class="buttons">
          <a class="button" href="/login">Log in</a>
          <a class="button is-primary" href="/signup">Sign up</a>
        </div>
      </div>
    </div>
  </div>
</nav>

<section class="section">
  <h1 class="title">Sign up</h1>

  {#if message}
    <p class="notification is-info">{message}</p>
  {/if}

  <form on:submit={handleSignup}>
    <div class="field">
      <label class="label" for="first-name">First Name</label>
      <input
        class="input"
        id="first-name"
        type="text"
        placeholder="Enter first name"
        bind:value={firstName}
        required
      />
    </div>
  
    <div class="field">
      <label class="label" for="last-name">Last Name</label>
      <input
        class="input"
        id="last-name"
        type="text"
        placeholder="Enter last name"
        bind:value={lastName}
        required
      />
    </div>
  
    <div class="field">
      <label class="label" for="email">Email</label>
      <input
        class="input"
        id="email"
        type="email"
        placeholder="Enter email"
        bind:value={email}
        required
      />
    </div>
  
    <div class="field">
      <label class="label" for="password">Password</label>
      <input
        class="input"
        id="password"
        type="password"
        placeholder="Enter password"
        bind:value={password}
        required
      />
    </div>
  
    <div class="field">
      <label class="label">Select Hospitals</label>
      <div class="box" style="max-height: 180px; overflow-y: auto;">
        {#each hospitals as hospital}
          <label class="checkbox mb-1">
            <input
              type="checkbox"
              checked={selectedHospitals.includes(hospital._id)}
              on:change={() => toggleHospital(hospital._id)}
            />
            &nbsp; {hospital.name}
          </label><br />
        {/each}
      </div>
    </div>
  
    <div class="field">
      <label class="label" for="profile-pic">Optional Profile Picture</label>
      <input
        id="profile-pic" 
        class="input"
        type="file"
        accept="image/*"
        bind:this={pictureInput}
      />
    </div>
  
    <div class="field mt-4">
      <button class="button is-link" type="submit">Submit</button>
    </div>
  </form>  
</section>
