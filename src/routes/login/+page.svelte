<script lang="ts">
    import { goto } from "$app/navigation";
    let email = "";
    let password = "";
    let message = "";
  
    async function handleLogin(event: Event) {
      event.preventDefault();
      message = "";
  
      try {
        const res = await fetch("http://localhost:3000/api/authenticate", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ email, password })
        });
  
        if (res.ok) {
          const result = await res.json();
          localStorage.setItem("jwt", result.token);
          goto("/dashboard");  // ✅ redirect after login
        } else {
          const text = await res.text();
          message = "❌ Login failed: " + text;
        }
      } catch (err) {
        message = "❌ Network error: " + err.message;
      }
    }
  </script>
  
  <section class="section">
    <div class="container">
      <h1 class="title">Log In</h1>
  
      {#if message}
        <div class="notification is-danger">
          {message}
        </div>
      {/if}
  
      <form on:submit={handleLogin}>
        <div class="field">
          <label class="label">Email</label>
          <div class="control">
            <input class="input" type="email" bind:value={email} required />
          </div>
        </div>
  
        <div class="field">
          <label class="label">Password</label>
          <div class="control">
            <input class="input" type="password" bind:value={password} required />
          </div>
        </div>
  
        <div class="field">
          <div class="control">
            <button class="button is-primary" type="submit">Login</button>
          </div>
        </div>
      </form>
    </div>
  </section>
  