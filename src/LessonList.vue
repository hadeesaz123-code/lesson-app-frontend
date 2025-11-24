<template>
  <!-- Outer wrapper for the lesson listing page -->
  <div class="lesson-list">

    <!-- Loading text while lessons are fetched -->
    <div v-if="loading" class="loading">Loading lessons…</div>

    <!-- Grid of lesson cards -->
    <ul v-else class="lessons">
      <!-- Single lesson card (clicking opens details modal) -->
      <li
        v-for="lesson in filteredLessons"
        :key="lesson._id || lesson.subject"
        class="lesson-card"
        @click="openDetails(lesson)"
      >
        <!-- Top image section -->
        <div class="card-image">
          <img
            v-if="lesson.image"
            :src="imageSrc(lesson.image)"
            :alt="lesson.subject"
          />
        </div>

        <!-- Text and metadata area -->
        <div class="card-body">
          <!-- Lesson subject/title -->
          <h3 class="title">{{ lesson.subject }}</h3>

          <!-- Short description -->
          <p class="desc">{{ lesson.description }}</p>

          <!-- Bottom row: price, spaces, and add button -->
          <div class="meta-row">

            <!-- Price and spaces info -->
            <div class="info">
              <span class="price">£{{ lesson.price }}</span>
              <span class="spaces">{{ lesson.spaces }} spaces</span>
            </div>

            <!-- Add to cart button (.stop stops opening the modal) -->
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

        <!-- Modal title -->
        <h2>{{ selectedLesson.subject }}</h2>

        <!-- Large image in modal -->
        <img
          v-if="selectedLesson.image"
          :src="imageSrc(selectedLesson.image)"
          class="detail-img"
        />

        <!-- Full description and data -->
        <p class="detail-desc">{{ selectedLesson.description }}</p>
        <p><strong>Price:</strong> £{{ selectedLesson.price }}</p>
        <p><strong>Location:</strong> {{ selectedLesson.location }}</p>
        <p><strong>Spaces left:</strong> {{ selectedLesson.spaces }}</p>

        <!-- Add to cart from modal -->
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

  // Props received from parent (App.vue)
  props: {
    // Search text typed by user in parent
    search: { type: String, default: "" }
  },

  // Local state for this component
  data() {
    return {
      lessons: [],        // All lessons loaded from backend
      loading: true,      // True while fetching lessons
      searchTimer: null,  // Timer id used for debounced search
      selectedLesson: null // Lesson currently shown in modal
    };
  },

  // Watchers react to change in props or data
  watch: {
    // When search prop changes, start a debounced search
    search(newValue) {
      this.debounceSearch(newValue);
    }
  },

  // Computed properties are derived from data/props
  computed: {
    // Filter lessons on the front-end based on search text
    filteredLessons() {
      // Convert search text to lowercase and trim spaces
      const q = this.search.toLowerCase().trim();

      // If there is no query, return all lessons
      if (!q) return this.lessons;

      // Otherwise filter by subject or description
      return this.lessons.filter(l =>
        (l.subject || "").toLowerCase().includes(q) ||
        (l.description || "").toLowerCase().includes(q)
      );
    }
  },

  methods: {
    // Fetch all lessons from backend API
    async fetchLessons() {
      try {
        // Send GET request to /lessons
        const res = await fetch(`${API}/lessons`);
        // Parse JSON body and store in lessons array
        this.lessons = await res.json();
      } catch (err) {
        // Log any error in console
        console.error("Error loading lessons:", err);
      } finally {
        // Always hide loading state after attempt
        this.loading = false;
      }
    },

    // Delay search until user stops typing
    debounceSearch(q) {
      // Clear previous timer if any
      clearTimeout(this.searchTimer);
      // Start a new timer that runs doSearch after 250ms
      this.searchTimer = setTimeout(() => this.doSearch(q), 250);
    },

    // Perform search on backend API
    async doSearch(q) {
      // If query is empty, simply reload all lessons
      if (!q) return this.fetchLessons();

      try {
        // Send GET request to /search with query
        const res = await fetch(`${API}/search?q=${encodeURIComponent(q)}`);
        // Replace lessons with filtered results from backend
        this.lessons = await res.json();
      } catch (err) {
        // Log search error
        console.error("Search failed:", err);
      }
    },

    // Build image URL (for now just return raw value or empty string)
    imageSrc(raw) {
      return raw || "";
    },

    // Add lesson to cart and decrease spaces by one
    addToCart(lesson) {
      // Emit event so parent (App.vue) can update its cart
      this.$emit("add-to-cart", {
        id: lesson._id || lesson.subject, // Unique identifier
        subject: lesson.subject,          // Title for display
        price: lesson.price,              // Price for cart
        quantity: 1                       // One unit each click
      });

      // Decrease local spaces count for this lesson
      lesson.spaces--;
    },

    // Open modal with lesson details
    openDetails(lesson) {
      this.selectedLesson = lesson;
    },

    // Close modal by clearing selectedLesson
    closeDetails() {
      this.selectedLesson = null;
    }
  },

  // Lifecycle hook: runs when component is inserted in the page
  mounted() {
    // Load lessons as soon as component is ready
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
