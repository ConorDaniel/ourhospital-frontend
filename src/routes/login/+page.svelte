<script lang="ts">
  import { goto } from "$app/navigation";

  const API_URL = import.meta.env.VITE_API_URL;

  let email = "";
  let password = "";
  let error = "";

  async function handleLogin() {
  error = "";
  try {
    const res = await fetch(`${API_URL}/api/authenticate`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email, password })
    });

    if (res.ok) {
      const result = await res.json();
      localStorage.setItem("token", result.token); // ✅ Save token
      goto("/dashboard");
    } else {
      const result = await res.json();
      error = result.error || "Login failed.";
    }
  } catch (err) {
    error = "Could not connect to server.";
  }
}
</script>

<style>
  .header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 1rem;
    border-bottom: 1px solid #e0e0e0;
  }

  .logo-container {
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  .logo {
    width: 100px;
    height: auto;
  }

  .title {
    font-size: 2rem;
    font-weight: bold;
    color: #00c4b3;
  }

  .form-container {
    margin: 3rem auto;
    max-width: 400px;
    padding: 2rem;
    border: 1px solid #ddd;
    border-radius: 8px;
  }

  .form-container input {
    width: 100%;
    padding: 0.6rem;
    margin: 0.5rem 0;
    border: 1px solid #ccc;
    border-radius: 4px;
  }

  .form-container button {
    margin-top: 1rem;
    width: 100%;
    padding: 0.8rem;
    background-color: #00c4b3;
    color: white;
    border: none;
    border-radius: 4px;
    font-weight: bold;
    cursor: pointer;
  }

  .error {
    color: red;
    margin-top: 1rem;
  }
</style>

<div class="header">
  <div class="logo-container">
    <img src="/images/logo.jpg" alt="Hospital Logo" class="logo" />
    <div class="title">OurHospital</div>
  </div>
</div>

<div class="form-container">
  <h2>Log in</h2>
  <form on:submit|preventDefault={handleLogin}>
    <input type="email" bind:value={email} placeholder="Email" required />
    <input type="password" bind:value={password} placeholder="Password" required />
    <button type="submit">Login</button>
  </form>
  {#if error}
    <div class="error">{error}</div>
  {/if}
</div>

