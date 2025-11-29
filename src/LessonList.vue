<template>
  // Outer wrapper for the lesson listing page
  <div class="lesson-list">

    //Loading text while lessons are fetched
    <div v-if="loading" class="loading">Loading lessons…</div>

   
    <ul v-else class="lessons">

      //Single lesson card (clicking opens details modal)
      <li
        v-for="lesson in filteredLessons"
        :key="lesson._id || lesson.subject"
        class="lesson-card"
        @click="openDetails(lesson)"
      >

        // Top image section 
        <div class="card-image">
          <img
            v-if="lesson.image"
            :src="imageSrc(lesson.image)"
            :alt="lesson.subject"
          />
        </div>

        <div class="card-body">

         
          <h3 class="title">{{ lesson.subject }}</h3>

        
          <p class="desc">{{ lesson.description }}</p>

          // Bottom row: price, spaces, and add button -->
          <div class="meta-row">

            // Price and spaces info
            <div class="info">
              <span class="price">£{{ lesson.price }}</span>
              <span class="spaces">{{ lesson.spaces }} spaces</span>
            </div>

            // Add to cart button (stop prevents modal from opening)
            <div class="actions">
              <button
                :disabled="lesson.spaces <= 0"
                @click.stop="addToCart(lesson)"
              >
                {{ lesson.spaces > 0 ? "Add to cart" : "Full" }}
              </button>
            </div>

          </div>
        </div>
      </li>
    </ul>

    <!-- Details modal, shown when a lesson is selected -->
    <div v-if="selectedLesson" class="modal-backdrop">

      <div class="modal">

       
        <h2>{{ selectedLesson.subject }}</h2>

        <!-- Large image in modal -->
        <img
          v-if="selectedLesson.image"
          :src="imageSrc(selectedLesson.image)"
          class="detail-img"
        />

        //Full description and data
        <p class="detail-desc">{{ selectedLesson.description }}</p>
        <p><strong>Price:</strong> £{{ selectedLesson.price }}</p>
        <p><strong>Location:</strong> {{ selectedLesson.location }}</p>
        <p><strong>Spaces left:</strong> {{ selectedLesson.spaces }}</p>

        // Add to cart from modal
        <button
          class="add-btn"
          :disabled="selectedLesson.spaces <= 0"
          @click="addToCart(selectedLesson)"
        >
          Add to cart
        </button>

        <!-- Close modal -->
        <button class="close-btn" @click="closeDetails">Close</button>

      </div>
    </div>

  </div>
</template>

<script>
// Choose API base URL depending on where the app is running
const API =
  window.location.hostname === "localhost"
    ? "http://localhost:3000"
    : "https://lesson-app-backend-7mte.onrender.com";

export default {
  // Component name
  name: "LessonList",

  // Props received from App.vue
  props: {
    // Search text typed by user in parent
    search: { type: String, default: "" }
  },

  // Local state for this component
  data() {
    return {
      lessons: [],         // All lessons loaded from backend
      loading: true,       // True while fetching lessons
      searchTimer: null,   // Timer used for debouncing search
      selectedLesson: null // Lesson currently shown in modal
    };
  },

  // Runs when data changes
watch: {
    // When the search prop changes, start debounced search
    search(newValue) {
      this.debounceSearch(newValue);
    }
  },

  // updates automatically when data changes
  computed: {
    // Filter lessons displayed on frontend
    filteredLessons() {

      // Convert search text to lowercase to match regardless of case
      const q = this.search.toLowerCase().trim();

      // If empty search box, show all lessons
      if (!q) return this.lessons;

      // Otherwise filter lessons by subject or description
      return this.lessons.filter(l =>
        (l.subject || "").toLowerCase().includes(q) ||
        (l.description || "").toLowerCase().includes(q)
      );
    }
  },

  methods: {

    // Load all lessons from backend
    async fetchLessons() {
      try {

        // Send GET request to /lessons API
        const res = await fetch(`${API}/lessons`);

        // Convert JSON response into array of lessons
        this.lessons = await res.json();

      } catch (err) {

        // Print error if fetch fails
        console.error("Error loading lessons:", err);

      } finally {

        // Hide loading message after request is finished
        this.loading = false;

      }
    },

    // Debounce prevents search from running too often
    debounceSearch(q) {

      // Remove previous timer if it exists
      clearTimeout(this.searchTimer);

      // Start new delay before search
      this.searchTimer = setTimeout(() => this.doSearch(q), 250);

    },

    // Perform backend-based search
    async doSearch(q) {

      // If search field is empty re-fetch all lessons
      if (!q) return this.fetchLessons();

      try {

        // Send request to backend search API
        const res = await fetch(`${API}/search?q=${encodeURIComponent(q)}`);

        // Replace displayed lessons with filtered results
        this.lessons = await res.json();

      } catch (err) {

        // Print error if search fails
        console.error("Search failed:", err);

      }
    },

    imageSrc(raw) {

  // If image already has https, use it
  if (raw.startsWith("http")) return raw;

  // If not, load it from backend
  return `https://lesson-app-backend-7mte.onrender.com${raw}`;
}
npmr
    // Add lesson to cart AND store original space count
    addToCart(lesson) {

      // send data so App.vue can update the cart
      this.$emit("add-to-cart", {
        id: lesson._id || lesson.subject, // Unique lesson identifier
        subject: lesson.subject,          // Lesson title
        price: lesson.price,              // Lesson price
        quantity: 1,                      // Quantity per click
        spaces: lesson.spaces             // Save space BEFORE decrease
      });

      // Immediately reduce displayed spaces in the UI
      lesson.spaces--;
    },

    // Open modal with lesson details
    openDetails(lesson) {
      this.selectedLesson = lesson;
    },

    // Close modal view
    closeDetails() {
      this.selectedLesson = null;
    }
  },

  // Lifecycle hook: runs when component loads
  mounted() {

    // Fetch lessons immediately on page load
    this.fetchLessons();

  }
};
</script>

<style scoped>
/* Container for lessons - keeps layout consistent */
.lesson-list {
  padding: 24px;
}

/* Grid layout for lessons */
.lessons {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 24px;
  padding: 0;
  list-style: none;
}

/* Individual card for each lesson */
.lesson-card {
  background: #ffffff;
  border-radius: 14px;
  box-shadow: 0 4px 12px rgba(15, 23, 42, 0.08);
  cursor: pointer;
  overflow: hidden;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}

/* Hover effect to lift the card slightly */
.lesson-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 18px rgba(15, 23, 42, 0.12);
}

/* Image wrapper */
.card-image img {
  width: 100%;
  height: 160px;
  object-fit: cover;
}

/* Body of the card */
.card-body {
  padding: 16px 16px 14px;
}

/* Lesson title styling */
.title {
  margin: 0 0 6px;
  font-size: 18px;
  font-weight: 600;
  color: #111827;
}

/* Lesson short description styling */
.desc {
  margin: 0 0 14px;
  font-size: 14px;
  line-height: 1.4;
  color: #6b7280;
}

/* Bottom row of card: price + button */
.meta-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* Price styling */
.price {
  display: block;
  font-weight: 600;
  font-size: 16px;
  color: #111827;
}

/* Spaces label */
.spaces {
  display: block;
  font-size: 12px;
  color: #9ca3af;
}

/* Add to cart button styling */
.actions button {
  background: #4b5563; /* neutral grey */
  color: #ffffff;
  border: none;
  padding: 8px 14px;
  font-size: 13px;
  border-radius: 999px;
  cursor: pointer;
  transition: background 0.15s ease, transform 0.1s ease;
}

/* Disabled add button */
.actions button:disabled {
  background: #d1d5db;
  cursor: default;
}

/* Hover on enabled button */
.actions button:hover:not(:disabled) {
  background: #374151;
  transform: translateY(-1px);
}

/* Full-page modal backdrop */
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(15, 23, 42, 0.35);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 50;
}

/* Modal content box */
.modal {
  background: #ffffff;
  padding: 24px 24px 20px;
  border-radius: 16px;
  width: 90%;
  max-width: 480px;
  box-shadow: 0 12px 30px rgba(15, 23, 42, 0.25);
}

/* Modal main image */
.detail-img {
  width: 100%;
  border-radius: 12px;
  margin: 12px 0 14px;
}

/* Modal description */
.detail-desc {
  font-size: 14px;
  color: #4b5563;
  margin-bottom: 12px;
}

/* Add button in modal */
.add-btn {
  width: 100%;
  background: #4b5563;
  color: #ffffff;
  padding: 10px 14px;
  border-radius: 999px;
  border: none;
  margin: 12px 0;
  cursor: pointer;
}
.add-btn:disabled {
  background: #d1d5db;
}

/* Close button in modal */
.close-btn {
  width: 100%;
  border-radius: 999px;
  padding: 10px 14px;
  border: 1px solid #d1d5db;
  background: #f9fafb;
  cursor: pointer;
}

/* Loading text */
.loading {
  padding: 40px 0;
  text-align: center;
  font-size: 16px;
  color: #4b5563;
}
</style>
