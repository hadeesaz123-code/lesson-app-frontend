<template>
  <div class="lesson-list">

    <div v-if="loading" class="loading">Loading lessons…</div>

    <ul v-else class="lessons">
      <li
        v-for="lesson in filteredLessons"
        :key="lesson._id || lesson.subject"
        class="lesson-card"
        @click="openDetails(lesson)"
      >

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

          <div class="meta-row">
            <div class="info">
              <span class="price">£{{ lesson.price }}</span>
              <span class="spaces">{{ lesson.spaces }} spaces</span>
            </div>

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

    <div v-if="selectedLesson" class="modal-backdrop">
      <div class="modal">

        <h2>{{ selectedLesson.subject }}</h2>

        <img
          v-if="selectedLesson.image"
          :src="imageSrc(selectedLesson.image)"
          class="detail-img"
        />

        <p class="detail-desc">{{ selectedLesson.description }}</p>
        <p><strong>Price:</strong> £{{ selectedLesson.price }}</p>
        <p><strong>Location:</strong> {{ selectedLesson.location }}</p>
        <p><strong>Spaces left:</strong> {{ selectedLesson.spaces }}</p>

        <button
          class="add-btn"
          :disabled="selectedLesson.spaces <= 0"
          @click="addToCart(selectedLesson)"
        >
          Add to cart
        </button>

        <button class="close-btn" @click="closeDetails">Close</button>

      </div>
    </div>

  </div>
</template>

<script>
// Pick the right backend address
const API =
  window.location.hostname === "localhost"
    ? "http://localhost:3000"
    : "https://lesson-app-backend-7mte.onrender.com";

export default {
  name: "LessonList",

  // Get search text from App.vue
  props: {
    search: { type: String, default: "" }
  },

  // Data for this page
  data() {
    return {
      lessons: [],
      loading: true,
      searchTimer: null,
      selectedLesson: null
    };
  },

  // Run when search text changes
  watch: {
    search(newValue) {
      this.debounceSearch(newValue);
    }
  },

  // Filter lessons on screen
  computed: {
    filteredLessons() {
      const q = this.search.toLowerCase().trim();

      if (!q) return this.lessons;

      return this.lessons.filter(l =>
        (l.subject || "").toLowerCase().includes(q) ||
        (l.description || "").toLowerCase().includes(q)
      );
    }
  },

  methods: {

    // Get all lessons
    async fetchLessons() {
      try {
        const res = await fetch(`${API}/lessons`);
        this.lessons = await res.json();
      } catch (err) {
        console.error("Load failed:", err);
      } finally {
        this.loading = false;
      }
    },

    // Delay search
    debounceSearch(q) {
      clearTimeout(this.searchTimer);
      this.searchTimer = setTimeout(() => this.doSearch(q), 250);
    },

    // Search on backend
    async doSearch(q) {
      if (!q) return this.fetchLessons();

      try {
        const res = await fetch(`${API}/search?q=${encodeURIComponent(q)}`);
        this.lessons = await res.json();
      } catch (err) {
        console.error("Search error:", err);
      }
    },

    // Build correct image path
    imageSrc(raw) {
      if (raw.startsWith("http")) return raw;
      return `https://lesson-app-backend-7mte.onrender.com${raw}`;
    },

    // Add item and reduce spaces
    addToCart(lesson) {
      this.$emit("add-to-cart", {
        id: lesson._id || lesson.subject,
        subject: lesson.subject,
        price: lesson.price,
        quantity: 1,
        spaces: lesson.spaces
      });

      lesson.spaces--;
    },

    // Show details popup
    openDetails(lesson) {
      this.selectedLesson = lesson;
    },

    // Close popup
    closeDetails() {
      this.selectedLesson = null;
    }
  },

  // Load when page opens
  mounted() {
    this.fetchLessons();
  }
};
</script>

<style scoped>
.lesson-list {
  padding: 24px;
}

.lessons {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 24px;
  padding: 0;
  list-style: none;
}

.lesson-card {
  background: #ffffff;
  border-radius: 14px;
  box-shadow: 0 4px 12px rgba(15, 23, 42, 0.08);
  cursor: pointer;
  overflow: hidden;
}

.lesson-card:hover {
  transform: translateY(-4px);
}

.card-image img {
  width: 100%;
  height: 160px;
  object-fit: cover;
}

.card-body {
  padding: 16px;
}

.title {
  margin: 0 0 6px;
  font-size: 18px;
}

.desc {
  font-size: 14px;
  color: #6b7280;
}

.meta-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.price {
  font-weight: 600;
}

.spaces {
  font-size: 12px;
  color: #9ca3af;
}

.actions button {
  background: #4b5563;
  color: white;
  padding: 8px 14px;
  border-radius: 999px;
  border: none;
}

.actions button:disabled {
  background: #d1d5db;
}

.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.4);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal {
  background: white;
  padding: 24px;
  max-width: 480px;
  border-radius: 16px;
}

.detail-img {
  width: 100%;
}

.add-btn, .close-btn {
  margin-top: 10px;
  width: 100%;
  padding: 10px;
}

.loading {
  text-align: center;
  padding: 30px;
}
</style>
