<script>
    // No JavaScript needed for now
  </script>
  
  <section class="section">
    <h1 class="title">About <em>OurHospital</em></h1>
  
    <div class="content">
      <p>
        <strong>OurHospital</strong> is a Svelte/Hapi project developed as part of the FullStack1 course at SETU. It provides a web-based hospital information and a rating platform with three levels of access:
      </p>
  
      <ul>
        <li>
          <strong>Public Users</strong><br />
          Anyone can:
          <ul>
            <li>Browse the list of hospitals from the MongoDB database</li>
            <li>View hospital details including bed count, staff, budget, and map location</li>
            <li>Submit a rating of the hospital across four categories</li>
            <li>Review comparative charts for hospital figures and ratings</li>
          </ul>
        </li>
  
        <li>
          <strong>Admins</strong><br />
          Admins can:
          <ul>
            <li>Access all hospitals in the system</li>
            <li>Add new hospitals to the master list</li>
            <li>Edit hospital details and associated metadata</li>
            <li>Admin privileges can only be granted by system-level admins</li>
          </ul>
        </li>
  
        <li>
          <strong>Registered Users</strong><br />
          Users can:
          <ul>
            <li>Select hospitals from the master list for contact management</li>
            <li>Add departments to their selected hospitals</li>
            <li>Add staff details to departments, including staff photos</li>
          </ul>
        </li>
      </ul>
  
      <p>
        Image uploads are handled on the backend and passed to Cloudinary for storage. The resulting URLs are stored and used in the frontend to dynamically render images across the app.
      </p>
  
      <hr />
  
      <p>Conor Leonard 20109017, based on class resources and with the help of ChatGPT 4o</p>
      <p>Here is a nice picture of a moose:</p>
  
      <figure class="image is-3by2 mt-5">
        <img
          src="https://res.cloudinary.com/dycaquyie/image/upload/v1747501851/samples/animals/reindeer.jpg"
          alt="Decorative Reindeer"
          style="border-radius: 0.5rem;"
        />
        <figcaption class="has-text-centered mt-2">
          A reminder to stay calm under pressure 🦌
        </figcaption>
      </figure>
  
      <hr />
  
      <h2 class="title is-5 mt-6">Technologies Used</h2>
      <ul>
        <li><strong>Frontend:</strong> SvelteKit, Bulma CSS, Leaflet (for maps), Frappe Charts</li>
        <li><strong>Backend:</strong> Hapi.js, MongoDB Atlas, JWT authentication</li>
        <li><strong>Cloud Services:</strong> Cloudinary (for image uploads)</li>
        <li><strong>Deployment:</strong> Node.js environment with Vite for development</li>
      </ul>
    </div>
  </section>
  