<script lang="ts">
    import { onMount, tick } from "svelte";
    import { page } from "$app/stores";
    import HospitalMap from "$lib/components/HospitalMap.svelte";
    import ImageRotator from "$lib/components/ImageRotator.svelte";

    let hospitals = [];
    let error = "";
    let searchTerm = "";

    // Get highlight target from query
    let highlightId = "";
    $: highlightId = $page.url.searchParams.get("highlight") || "";

    // Filter hospitals by search term
    $: filteredHospitals = hospitals.filter(h =>
        h.name.toLowerCase().includes(searchTerm.trim().toLowerCase())
    );

    onMount(async () => {
  try {
    const res = await fetch(`${import.meta.env.VITE_API_URL}/api/hospitals`);
    if (!res.ok) {
      error = "Failed to load hospitals: " + (await res.text());
      return;
    }

    const hospitalList = await res.json();

    // Fetch average ratings per hospital
    const ratings = await Promise.all(
      hospitalList.map(async (h) => {
        const r = await fetch(`${import.meta.env.VITE_API_URL}/api/hospitals/${h._id}/ratings/average`);
        return r.ok ? await r.json() : {};
      })
    );

    // Merge hospital + average rating
    hospitals = hospitalList.map((h, i) => ({
      ...h,
      averageRating: ratings[i] ?? {}
    }));

    // Scroll highlight if needed
    await tick();
    if (highlightId) {
      const el = document.getElementById(`hospital-${highlightId}`);
      if (el) {
        el.scrollIntoView({ behavior: "smooth", block: "center" });
        el.classList.add("highlight");
        setTimeout(() => el.classList.remove("highlight"), 3000);
      }
    }
  } catch (err) {
    error = "Network error: " + err.message;
  }
});

    function handleDeptClick() {
        alert("Please log in or sign up to access department details.");
    }

    function stars(score) {
  const full = Math.round(score ?? 0);
  const empty = 5 - full;
  return (
    `<span style="color:gold;">${"★".repeat(full)}</span>` +
    `<span style="color:#ddd;">${"★".repeat(empty)}</span>`
  );
}

</script>

<nav class="navbar sticky-navbar" aria-label="main navigation">
  <div class="navbar-brand">
        <img src="/images/logo.jpg" alt="Hospital logo" style="max-height: 150px; margin: 30px;" />
        <span class="title is-1 has-text-primary" style="margin-left: 30px;">OurHospital</span>
    </div>

    <div class="navbar-menu">
        <div class="navbar-end">
            <div class="navbar-item">
                <div class="buttons">
                    <a href="/about" class="button is-light">About</a>
                    <a class="button" href="/login">Log in</a>
                    <a class="button" href="/signup">Sign up</a>
                    <a class="button" href="/charts">📊 Charts</a>
                </div>
            </div>
        </div>
    </div>
</nav>

<section
  class="fixed-hero"
  style="background-image: url('https://res.cloudinary.com/dycaquyie/image/upload/c_fill,w_1600,h_600,f_auto,q_auto,e_blur:30/samples/imagecon-group');"
>
  <div class="hero-body has-text-centered">
    <div class="container">
      <h1 class="title has-text-white is-2" style="margin-left: 40px; text-shadow: 2px 2px 2px black, 0 0 5px black, 1px 1px 2px rgba(0,0,0,0.8);">
        Find us, Rate us!
      </h1>
      <h5 class="title has-text-warning is-5" style="margin-left: 20px; text-shadow: 1px 1px 2px black, 0 0 3px black, 1px 1px 1px rgba(0,0,0,0.7);">
        Log in for info on Departments and Staff
      </h5>
          </div>
  </div>
</section>

<section class="section">
    <h2 class="title is-4">Browse Hospitals</h2>

    <div class="field mb-5" style="max-width: 400px;">
      <label class="label" for="hospital-search">Search hospitals by name</label>
      <div class="control">
        <input
          id="hospital-search"
          class="input"
          type="text"
          placeholder="e.g. Mater"
          bind:value={searchTerm}
        />
      </div>
    </div>    

    {#if error}
        <p class="notification is-danger">{error}</p>
    {:else if filteredHospitals.length === 0}
        <p class="notification is-warning">No matching hospitals found.</p>
    {:else}
        {#each filteredHospitals as hospital}
            <div class="box mb-5" id={`hospital-${hospital._id}`}>
                <!-- Row 1: Name and Button -->
                <div class="columns is-vcentered is-mobile is-multiline mb-2">
                    <div class="column is-8">
                        <h2 class="title is-4 has-text-weight-bold is-uppercase">
                            {hospital.name}
                        </h2>
                        <p class="mt-1">
                            🛏️ <strong>Beds:</strong> {hospital.bedCount ?? "n/a"} &nbsp;&nbsp;
                            👨‍⚕️ <strong>Staff:</strong> {hospital.staffCount ?? "n/a"} &nbsp;&nbsp;
                            💶 <strong>Budget:</strong> €{hospital.budget?.toFixed(2) ?? "n/a"}M
                        </p>
                    </div>
                    <div class="column is-4 has-text-right">
                        <button class="button is-info" on:click={handleDeptClick}>
                            Departments
                        </button>
                    </div>
                </div>

                <!-- Row 2: Image, Rate Button, Map -->
                <div class="columns is-variable is-1 is-multiline">
                    <div class="column is-one-third">
                        {#if hospital.imageUrls && hospital.imageUrls.length > 1}
                          <ImageRotator images={hospital.imageUrls} />
                        {:else if hospital.imageUrls && hospital.imageUrls.length === 1}
                          <figure class="image is-3by2">
                            <img src={hospital.imageUrls[0]} alt="Hospital Image" style="border-radius: 0.5rem;" />
                          </figure>
                        {:else}
                          <ImageRotator
                            images={[
                              "/images/hospitals/mater-1a.jpg",
                              "/images/hospitals/mater-1b.jpg",
                              "/images/hospitals/mater-1c.jpg"
                            ]}
                          />
                        {/if}
                      
                        {#if hospital.averageRating}
                          <div class="has-text-left mt-3" style="padding-left: 10px;">
                            <div><strong>Care:</strong> {@html stars(hospital.averageRating.care)}</div>
                            <div><strong>Cleanliness:</strong> {@html stars(hospital.averageRating.cleanliness)}</div>
                            <div><strong>Friendliness:</strong> {@html stars(hospital.averageRating.friendliness)}</div>
                            <div><strong>Food:</strong> {@html stars(hospital.averageRating.food)}</div>
                          </div>
                        {:else}
                          <p class="has-text-grey mt-3 has-text-centered">No ratings yet</p>
                        {/if}
                      
                        <div class="has-text-centered mt-3">
                          <a
                            class="button is-link is-light is-medium"
                            style="font-size: 1.1rem; padding: 0.75rem 1.5rem;"
                            href={`/rate?hospitalId=${hospital._id.toString()}`}
                          >
                            ⭐ Rate this hospital
                          </a>
                        </div>
                      </div>                      

                    <div class="column is-two-thirds">
                        <div class="map-wrapper">
                            {#key hospital._id}
                                <HospitalMap lat={hospital.latitude} lng={hospital.longitude} />
                            {/key}
                        </div>
                    </div>
                </div>
            </div>
        {/each}
    {/if}
</section>

<style>
	.map-wrapper {
	  height: 300px;
	  position: relative;
	  border-radius: 0.5rem;
	  border: 1px solid #ddd;
	  overflow: hidden;
	}
  
	.highlight {
	  border: 3px solid #38b000;
	  box-shadow: 0 0 15px rgba(56, 176, 0, 0.5);
	  transition: all 0.3s ease-in-out;
	}
  
	.fixed-hero {
	  height: 300px;
	  background-size: cover;
	  background-position: center;
	  margin-top: 10px; /* creates spacing below navbar */
	}
  
	.sticky-navbar {
	  position: sticky;
	  top: 0;
	  z-index: 999;
	  background-color: white;
	  border-bottom: 1px solid #ddd;
	}
  </style>  
