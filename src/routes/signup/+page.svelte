<script lang="ts">
  import { onMount } from "svelte";

  let firstName = "";
  let lastName = "";
  let email = "";
  let password = "";
  let message = "";

  let hospitals = [];
  let selectedHospitals: string[] = [];

  onMount(async () => {
    try {
      const response = await fetch("http://localhost:3000/api/hospitals");
      if (response.ok) {
        hospitals = await response.json();
      } else {
        message = "❌ Failed to load hospital list";
      }
    } catch (err) {
      message = "❌ Network error: " + err.message;
    }
  });

  async function handleSignup(event: Event) {
    event.preventDefault();
    message = "";

    try {
      const response = await fetch("http://localhost:3000/api/users", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          firstName,
          lastName,
          email,
          password,
          hospitals: selectedHospitals
        })
      });

      if (response.ok) {
        message = "✅ Sign-up successful!";
        // Optional: redirect to login page
        // window.location.href = "/login";
      } else {
        const error = await response.text();
        message = "❌ Sign-up failed: " + error;
      }
    } catch (err) {
      message = "❌ Network error: " + err.message;
    }
  }
</script>

<!-- ✅ NAVBAR -->
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

<!-- ✅ SIGNUP FORM -->
<section class="section">
  <h1 class="title">Sign up</h1>

  {#if message}
    <p class="notification is-info">{message}</p>
  {/if}

  <form on:submit={handleSignup}>
    <div class="field">
      <label class="label">Name</label>
      <div class="field is-horizontal">
        <div class="field-body">
          <div class="field">
            <input
              class="input"
              type="text"
              placeholder="Enter first name"
              bind:value={firstName}
              required
            />
          </div>
          <div class="field">
            <input
              class="input"
              type="text"
              placeholder="Enter last name"
              bind:value={lastName}
              required
            />
          </div>
        </div>
      </div>
    </div>

    <div class="field">
      <label class="label">Email</label>
      <input
        class="input"
        type="email"
        placeholder="Enter email"
        bind:value={email}
        required
      />
    </div>

    <div class="field">
      <label class="label">Password</label>
      <input
        class="input"
        type="password"
        placeholder="Enter password"
        bind:value={password}
        required
      />
    </div>

    <div class="field">
      <label class="label">Select Hospitals</label>
      <div class="control">
        <div class="select is-multiple is-fullwidth">
          <select bind:value={selectedHospitals} multiple required size="5">
            {#each hospitals as hospital}
              <option value={hospital._id}>{hospital.name}</option>
            {/each}
          </select>
        </div>
      </div>
    </div>

    <div class="field">
      <button class="button is-link" type="submit">Submit</button>
    </div>
  </form>
</section>
