<script lang="ts">
  import { onMount } from "svelte";
  import { page } from "$app/stores";
  import { get } from "svelte/store";

  let hospitalId = get(page).url.searchParams.get("hospitalId");
  let hospitalName = "";
  let departments = [];
  let title = "";
  let deptLocation = "";
  let error = "";

  onMount(async () => {
    if (!hospitalId) {
      error = "No hospital selected.";
      return;
    }

    await fetchDepartments();
  });

  async function fetchDepartments() {
    const res = await fetch(`${import.meta.env.VITE_API_URL}/api/hospitals/${hospitalId}/departments`);
    if (res.ok) {
      departments = await res.json();
    } else {
      error = "Failed to load departments.";
    }

    const hospitalRes = await fetch(`${import.meta.env.VITE_API_URL}/api/hospitals/${hospitalId}`, {
      headers: {
        Authorization: `Bearer ${localStorage.getItem("jwt")}`
      }
    });
    if (hospitalRes.ok) {
      const hospital = await hospitalRes.json();
      hospitalName = hospital.name;
    }
  }

  async function handleSubmit(event: Event) {
    event.preventDefault();
    error = "";

    if (!title || !deptLocation) {
      error = "Both fields are required.";
      return;
    }

    const token = localStorage.getItem("jwt");
    if (!token) {
      error = "Not logged in.";
      return;
    }

    const newDepartment = { title, deptLocation };

    const res = await fetch(`${import.meta.env.VITE_API_URL}/api/hospitals/${hospitalId}/departments`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${token}`
      },
      body: JSON.stringify(newDepartment)
    });

    if (res.ok) {
      title = "";
      deptLocation = "";
      await fetchDepartments();
    } else {
      const result = await res.json().catch(() => null);
      error = result?.message || "Failed to add department.";
    }
  }

  async function deleteDepartment(departmentId: string) {
    const confirmDelete = confirm("Are you sure you want to delete this department?");
    if (!confirmDelete) return;

    const token = localStorage.getItem("jwt");
    const res = await fetch(`${import.meta.env.VITE_API_URL}/api/departments/${departmentId}`, {
      method: "DELETE",
      headers: {
        Authorization: `Bearer ${token}`
      }
    });

    if (res.ok) {
      await fetchDepartments();
    } else {
      error = "Failed to delete department.";
    }
  }
</script>

<section class="section">
  <h1 class="title">Departments</h1>

  <!-- Hospital title and back button -->
  <div class="level mb-4">
    <div class="level-left">
      <h2 class="title is-4">{hospitalName}</h2>
    </div>
    <div class="level-right">
      <a class="button is-light" href="/dashboard">
        <span class="icon"><i class="fas fa-arrow-left"></i></span>
        <span>Back to Hospital List</span>
      </a>
    </div>
  </div>

  {#if error}
    <p class="notification is-danger">{error}</p>
  {/if}

  {#if departments.length > 0}
    {#each departments as dept}
      <div class="box box-link-hover-shadow">
        <div class="level">
          <div class="level-left">
            <h2 class="title is-4">{dept.title}</h2>
          </div>
          <div class="level-right">
            <p class="has-text-right"><strong>Location:</strong> {dept.deptLocation}</p>
          </div>
        </div>

        <div class="buttons">
          <a href={`/staff?departmentId=${dept._id}`} class="button is-link is-light">
            <span class="icon is-small"><i class="fas fa-users"></i></span>
            <span>Staff</span>
          </a>
          <button
            class="button is-danger is-light is-small"
            title="Delete Department"
            on:click={() => deleteDepartment(dept._id)}
          >
            <span class="icon is-small"><i class="fas fa-trash"></i></span>
            <span>Delete</span>
          </button>
        </div>
      </div>
    {/each}
  {:else}
    <p>No departments found.</p>
  {/if}

  <hr />

  <h2 class="title is-5">Add Department</h2>
  <form on:submit={handleSubmit}>
    <div class="field">
      <label class="label" for="title">Title</label>
      <div class="control">
        <input class="input" id="title" type="text" bind:value={title} required />
      </div>
    </div>

    <div class="field">
      <label class="label" for="deptLocation">Location</label>
      <div class="control">
        <input class="input" id="deptLocation" type="text" bind:value={deptLocation} required />
      </div>
    </div>

    <div class="field">
      <button class="button is-primary" type="submit">Add Department</button>
    </div>
  </form>
</section>

<style>
  .box-link-hover-shadow:hover {
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    cursor: pointer;
  }
</style>
