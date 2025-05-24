<script lang="ts">
  import { onMount, tick } from "svelte";
  import { Chart } from "frappe-charts";

  let hospitals = [];
  let chartData = [];

  // DOM refs for chart targets
  let bedsChartEl;
  let staffChartEl;
  let budgetChartEl;
  let ratingChartEl;

  onMount(async () => {
    try {
      const res = await fetch(`${import.meta.env.VITE_API_URL}/api/hospitals`);
      hospitals = await res.json();

      const ratings = await Promise.all(
        hospitals.map(async (h) => {
          const r = await fetch(`${import.meta.env.VITE_API_URL}/api/hospitals/${h._id}/ratings/average`);
          return r.ok ? await r.json() : {};
        })
      );

      chartData = hospitals.map((h, i) => ({
        name: h.name,
        beds: h.bedCount,
        staff: h.staffCount,
        budget: h.budget,
        care: ratings[i]?.care ?? 0,
        cleanliness: ratings[i]?.cleanliness ?? 0,
        friendliness: ratings[i]?.friendliness ?? 0,
        food: ratings[i]?.food ?? 0,
      }));

      await tick();

      if (bedsChartEl) {
        new Chart(bedsChartEl, {
          title: "Beds per Hospital",
          type: "bar",
          data: {
            labels: chartData.map(h => h.name),
            datasets: [{ values: chartData.map(h => h.beds || 0) }]
          }
        });
      }

      if (staffChartEl) {
        new Chart(staffChartEl, {
          title: "Staff per Hospital",
          type: "bar",
          data: {
            labels: chartData.map(h => h.name),
            datasets: [{ values: chartData.map(h => h.staff || 0) }]
          }
        });
      }

      if (budgetChartEl) {
        new Chart(budgetChartEl, {
          title: "Budget (€M) per Hospital",
          type: "bar",
          data: {
            labels: chartData.map(h => h.name),
            datasets: [{ values: chartData.map(h => h.budget || 0) }]
          }
        });
      }

      if (ratingChartEl) {
        new Chart(ratingChartEl, {
          title: "Average Care Ratings",
          type: "bar",
          data: {
            labels: chartData.map(h => h.name),
            datasets: [{ values: chartData.map(h => h.care || 0) }]
          },
          colors: ["#21ba45"]
        });
      }
    } catch (err) {
      console.error("Chart loading failed:", err);
    }
  });
</script>

<style>
  .chart-container {
    max-width: 100%;
    overflow-x: auto;
    margin-bottom: 2rem;
    padding: 1rem;
    border: 1px solid #ddd;
    border-radius: 6px;
    background-color: #fafafa;
  }

  .title {
    margin-bottom: 2rem;
  }
</style>

  
<section class="section">
  <h1 class="title">Hospital Charts</h1>

  <div class="chart-container my-5">
    <div bind:this={bedsChartEl}></div>
  </div>

  <div class="chart-container my-5">
    <div bind:this={staffChartEl}></div>
  </div>

  <div class="chart-container my-5">
    <div bind:this={budgetChartEl}></div>
  </div>

  <div class="chart-container my-5">
    <div bind:this={ratingChartEl}></div>
  </div>
</section>
