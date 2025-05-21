<script lang="ts">
  import { onMount } from "svelte";
  import HospitalMap from "$lib/components/HospitalMap.svelte";
  import ImageRotator from "$lib/components/ImageRotator.svelte";

  let hospitals = [];
  let error = "";
  let userEmail = "";
  let userRole = "";
  let pictureUrl = "";

  onMount(async () => {
    const token = localStorage.getItem("token");

    if (!token) {
      error = "Not logged in.";
      return;
    }

    try {
      const res = await fetch(`${import.meta.env.VITE_API_URL}/api/users/me`, {
        headers: { Authorization: `Bearer ${token}` },
      });

      if (res.ok) {
        const user = await res.json();
        userEmail = user.email;
        userRole = user.role;
        pictureUrl = user.pictureUrl?.trim() ||
          "https://res.cloudinary.com/dycaquyie/image/upload/v1747570490/Screenshot_2025-05-18_at_13.13.48_dywns0.png";

        if (user.role === "admin") {
          const hospitalRes = await fetch(`${import.meta.env.VITE_API_URL}/api/hospitals`, {
            headers: { Authorization: `Bearer ${token}` },
          });

          if (hospitalRes.ok) {
            hospitals = await hospitalRes.json();
          } else {
            error = "Failed to load hospitals for admin.";
          }
        } else {
          hospitals = user.hospitals ?? [];
        }
      } else {
        error = "Failed to load user info.";
      }
    } catch (err) {
      error = "Network error: " + err.message;
    }
  });

  const deleteHospital = async (id: string) => {
    const token = localStorage.getItem("token");

    if (!token) {
      error = "Not logged in.";
      return;
    }

    const confirmDelete = confirm("Are you sure you want to delete this hospital?");
    if (!confirmDelete) return;

    try {
      const res = await fetch(`http://localhost:3000/api/hospitals/${id}`, {
        method: "DELETE",
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });

      if (res.ok) {
        hospitals = hospitals.filter((h) => h._id !== id);
      } else {
        error = `Delete failed: ${await res.text()}`;
      }
    } catch (err) {
      error = `Network error: ${err.message}`;
    }
  };

  function handleLogout() {
    localStorage.removeItem("token");
    window.location.href = "/";
  }
</script>

<style>
  .map-wrapper {
    height: 300px;
    position: relative;
    border-radius: 0.5rem;
    border: 1px solid #ddd;
    overflow: hidden;
    z-index: 1;
  }

  .user-info {
    display: flex;
    align-items: center;
    margin-right: 1.5rem;
  }

  .user-info img {
    height: 40px;
    width: 40px;
    border-radius: 50%;
    margin-right: 0.75rem;
    margin-left: 40px;
  }

  .sticky-header {
    position: sticky;
    top: 0;
    z-index: 1100;
    background-color: white;
    border-bottom: 1px solid #ddd;
  }

  :global(.leaflet-container) {
    z-index: 1 !important;
  }
</style>

<section class="section">
  <div class="sticky-header">
    <nav class="level mb-0">
      <div class="level-left">
        <div class="level-item">
          <img src="/images/logo.jpg" alt="Logo" style="height: 50px; margin-right: 10px" />
          <span class="title is-4 has-text-primary">ourHospital</span>
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
          {#if userRole === "admin"}
            <a href="/admins/master-list" class="button is-primary">➕ Add Hospital</a>
          {/if}
          <button class="button is-danger" on:click={handleLogout}>Logout</button>
        </div>
      </div>
    </nav>

    <h1 class="title px-4 py-3">Hospital Dashboard</h1>
  </div>

  {#if error}
    <p class="notification is-danger">{error}</p>
  {:else if hospitals.length === 0}
    <p class="notification is-warning">No hospitals found for this user.</p>
  {:else}
    {#each hospitals as hospital}
      <div class="box mb-5">
        <div class="columns is-vcentered is-mobile is-multiline mb-2">
          <div class="column is-8">
            <h2 class="title is-4 has-text-weight-bold is-uppercase">
              {hospital.name}
            </h2>
          </div>
          <div class="column is-4 has-text-right">
            <div class="buttons is-right">
              <a href={`/departments?hospitalId=${hospital._id}`} class="button is-info">
                Departments
              </a>
              {#if userRole === "admin"}
                <a href={`/admins/edit-hospital?id=${hospital._id}`} class="button is-warning">
                  ✏️ Edit
                </a>
              {/if}
              <button
                class="button is-danger"
                disabled={hospital.departments && hospital.departments.length > 0}
                on:click={() => deleteHospital(hospital._id)}
              >
                Delete
              </button>
            </div>
          </div>          
        </div>

        <div class="columns is-variable is-1 is-multiline">
          <div class="column is-one-third">
            <ImageRotator
              images={[
                "/images/hospitals/mater-1a.jpg",
                "/images/hospitals/mater-1b.jpg",
                "/images/hospitals/mater-1c.jpg"
              ]}
            />
          </div>
          <div class="column is-two-thirds">
            <div class="map-wrapper">
              <HospitalMap lat={hospital.latitude} lng={hospital.longitude} />
            </div>
          </div>
        </div>
      </div>
    {/each}
  {/if}
</section>
