<script lang="ts">
  import { onMount } from "svelte";
  import { page } from "$app/stores";
  import { get } from "svelte/store";

  let pictureInput: HTMLInputElement;
  let departmentId = get(page).url.searchParams.get("departmentId");

  let departmentName = "";
  let staffList = [];
  let name = "";
  let role = "";
  let vignette = "";
  let error = "";

  onMount(async () => {
    if (!departmentId) {
      error = "No department selected.";
      return;
    }

    await fetchStaff();
  });

  async function fetchStaff() {
    try {
      const res = await fetch(`http://localhost:3000/api/departments/${departmentId}/staff`);
      if (res.ok) {
        staffList = await res.json();
      } else {
        error = "Failed to load staff list.";
      }

      const deptRes = await fetch(`http://localhost:3000/api/departments/${departmentId}`, {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`
        }
      });
      if (deptRes.ok) {
        const dept = await deptRes.json();
        departmentName = dept.title;
      }
    } catch (err) {
      error = "Error fetching staff or department: " + err.message;
    }
  }

  async function handleSubmit(event: Event) {
    event.preventDefault();
    error = "";

    if (!name || !role || !vignette) {
      error = "Please complete all fields.";
      return;
    }

    let pictureUrl = "";
    if (pictureInput?.files?.length) {
      const file = pictureInput.files[0];
      const formData = new FormData();
      formData.append("file", file);

      try {
        const uploadRes = await fetch("http://localhost:3000/api/images/upload", {
          method: "POST",
          body: formData,
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token") || ""}`
          }
        });

        if (uploadRes.ok) {
          const result = await uploadRes.json();
          pictureUrl = result.secure_url;
        } else {
          const errMsg = await uploadRes.text();
          error = `❌ Image upload failed: ${errMsg}`;
          return;
        }
      } catch (err) {
        error = "❌ Network error during image upload: " + err.message;
        return;
      }
    }

    try {
      const token = localStorage.getItem("token");

      const res = await fetch(`http://localhost:3000/api/departments/${departmentId}/staff`, {
        method: "POST",
        headers: {
          Authorization: `Bearer ${token}`,
          "Content-Type": "application/json"
        },
        body: JSON.stringify({
          name,
          role,
          vignette,
          pictureUrl
        })
      });

      if (res.ok) {
        name = "";
        role = "";
        vignette = "";
        pictureInput.value = "";
        await fetchStaff();
      } else {
        const result = await res.json().catch(() => null);
        error = result?.message || "Failed to add staff member.";
      }
    } catch (err) {
      error = "Failed to submit staff member: " + err.message;
    }
  }

  async function deleteStaff(staffId: string) {
    const confirmed = confirm("Are you sure you want to delete this staff member?");
    if (!confirmed) return;

    const token = localStorage.getItem("token");
    const res = await fetch(`http://localhost:3000/api/staff/${staffId}`, {
      method: "DELETE",
      headers: {
        Authorization: `Bearer ${token}`
      }
    });

    if (res.ok) {
      await fetchStaff();
    } else {
      error = "Failed to delete staff member.";
    }
  }
</script>

<section class="section">
  <h1 class="title">Staff</h1>

  {#if departmentName}
    <h2 class="subtitle">Department: {departmentName}</h2>
  {/if}

  {#if error}
    <p class="notification is-danger">{error}</p>
  {/if}

  {#if staffList.length > 0}
    <div class="columns is-multiline">
      {#each staffList as staff}
        <div class="column is-one-third">
          <div class="box">
            {#if staff.pictureUrl}
              <figure class="image mb-2">
                <img src={staff.pictureUrl} alt="Staff Picture" style="max-height: 120px; border-radius: 8px;" />
              </figure>
            {/if}
            <p><strong>{staff.name}</strong></p>
            <p><em>{staff.role}</em></p>
            <p>{staff.vignette}</p>
            <button
              class="button is-danger is-light is-small mt-2"
              on:click={() => deleteStaff(staff._id)}
            >
              Delete
            </button>
          </div>
        </div>
      {/each}
    </div>
  {:else}
    <p>No staff members found.</p>
  {/if}

  <hr />

  <h2 class="title is-5">Add Staff Member</h2>
  <form on:submit={handleSubmit}>
    <div class="field">
      <label class="label" for="name">Name</label>
      <input class="input" id="name" type="text" bind:value={name} required />
    </div>

    <div class="field">
      <label class="label" for="role">Role</label>
      <input class="input" id="role" type="text" bind:value={role} required />
    </div>

    <div class="field">
      <label class="label" for="vignette">Vignette</label>
      <textarea class="textarea" id="vignette" bind:value={vignette} required />
    </div>

    <div class="field">
      <label class="label" for="picture">Staff Picture (optional)</label>
      <input class="input" id="picture" type="file" accept="image/*" bind:this={pictureInput} />
    </div>

    <div class="field">
      <button class="button is-primary" type="submit">Add Staff</button>
    </div>
  </form>

  <div class="mt-4">
    <a href="/dashboard" class="button is-light">← Back to Dashboard</a>
  </div>
</section>
