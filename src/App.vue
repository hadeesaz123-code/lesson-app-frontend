<script>
// Load LessonList file
import LessonList from "./LessonList.vue";

// Pick the right backend address
const API =
  window.location.hostname === "localhost"
    ? "http://localhost:3000"
    : "https://lesson-app-backend-7mte.onrender.com";

export default {

  // Use LessonList on this page
  components: { LessonList },

  // Data used on the page
  data() {
    return {
      searchQuery: "",     // Search text
      showCart: false,     // Show cart page or not
      cart: [],            // Items in cart
      name: "",            // Name box
      phone: "",           // Phone box
      email: "",           // Email box
      showAuthModal: false,// Show login box
      authMode: "login",   // Login or register
      authEmail: "",       // Login email
      authPassword: "",    // Login password
      authName: "",        // Register name
      currentUser: null    // Logged in user
    };
  },

  // Runs when page loads
  mounted() {

    // Get user from browser
    const saved = localStorage.getItem("user");

    // If user saved, load it
    if (saved) {
      this.currentUser = JSON.parse(saved);
    }
  },

  computed: {

    // Check name letters only
    validName() {
      return /^[A-Za-z\s]+$/.test(this.name);
    },

    // Check phone numbers only
    validPhone() {
      return /^\d+$/.test(this.phone);
    },

    // Check email looks right
    validEmail() {
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(this.email);
    },

    // Allow order only if all ok
    canCheckout() {
      return (
        this.cart.length > 0 &&
        this.validName &&
        this.validPhone &&
        this.validEmail
      );
    },

    // Add all cart prices
    totalPrice() {
      return this.cart.reduce((total, item) => total + item.price, 0);
    }
  },

  methods: {

    // Add item to cart
    onAddToCart(item) {
      this.cart.push(item);
    },

    // Show or hide cart page
    toggleCart() {
      this.showCart = !this.showCart;
    },

    // Remove from cart
    removeFromCart(index) {
      this.cart.splice(index, 1);
      if (this.cart.length === 0) this.showCart = false;
    },

    // Go back home
    goHome() {
      this.showCart = false;
      this.searchQuery = "";
      window.scrollTo(0, 0);
    },

    // Open login box
    openAuth(mode) {
      this.authMode = mode;
      this.showAuthModal = true;
      this.authEmail = "";
      this.authPassword = "";
      this.authName = "";
    },

    // Close login box
    closeAuth() {
      this.showAuthModal = false;
    },

    // Login or register user
    async submitAuth() {

      // Pick path
      const endpoint =
        this.authMode === "login" ? "/login" : "/register";

      // Make user data
      const payload = {
        email: this.authEmail,
        password: this.authPassword,
        name: this.authMode === "register" ? this.authName : undefined
      };

      try {

        // Send to backend
        const res = await fetch(`${API}${endpoint}`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(payload)
        });

        // Get reply
        const data = await res.json();

        // Show error
        if (!res.ok) {
          alert(data.error || "Login failed");
          return;
        }

        // Save user
        this.currentUser = data.user || {
          name: payload.name,
          email: payload.email
        };

        // Save user in browser
        localStorage.setItem("user", JSON.stringify(this.currentUser));

        // Show message
        alert(data.message);
        this.closeAuth();

      } catch (error) {
        alert("No connection");
      }
    },

    // Logout user
    logout() {
      this.currentUser = null;
      localStorage.removeItem("user");
      alert("Logged out");
    },

    // Save order and reduce spaces
    async confirmOrder() {

      // Stop if wrong info
      if (!this.canCheckout) return;

      // Make order info
      const payload = {
        name: this.name,
        phone: this.phone,
        email: this.email,

        // Send lesson ids
        items: this.cart.map(item => ({
          lessonId: item.id,
          quantity: 1
        }))
      };

      // Send order
      await fetch(`${API}/orders`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
      });

      // Change spaces in database
      for (const item of this.cart) {

        // Take one space
        const newSpaces = item.spaces - 1;

        // Save updated number
        await fetch(`${API}/lessons/${item.id}`, {
          method: "PUT",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ spaces: newSpaces })
        });
      }

      // Show success
      alert("Order done!");

      // Clear cart
      this.cart = [];
      this.showCart = false;
    }
  }
};
</script>

<style>
body,
html {
  margin: 0;
  padding: 0;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  background: #f3f4f6;
}

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.header {
  background: #ffffff;
  border-bottom: 1px solid #e5e7eb;
  padding: 16px 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.header-title h1 {
  margin: 0;
  font-size: 22px;
  font-weight: 600;
  color: #111827;
}

.header-title p {
  margin: 4px 0 0;
  font-size: 14px;
  color: #6b7280;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 10px;
}

.search-input {
  padding: 8px 10px;
  border-radius: 999px;
  border: 1px solid #d1d5db;
  font-size: 14px;
  min-width: 180px;
  background: #f9fafb;
}

.auth-btn {
  padding: 8px 12px;
  border-radius: 999px;
  border: 1px solid #d1d5db;
  background: #ffffff;
  font-size: 14px;
  cursor: pointer;
}

.auth-user {
  display: flex;
  align-items: center;
  gap: 10px;
  font-weight: 500;
}

.cart-toggle {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 8px 12px;
  border-radius: 999px;
  border: 1px solid #d1d5db;
  background: #f9fafb;
  font-size: 14px;
  cursor: pointer;
}

.cart-toggle:disabled {
  opacity: 0.5;
  cursor: default;
}

.cart-label {
  color: #111827;
}

.cart-badge {
  min-width: 20px;
  height: 20px;
  padding: 0 6px;
  border-radius: 999px;
  background: #4b5563;
  color: #ffffff;
  font-size: 12px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.layout {
  flex: 1;
  max-width: 960px;
  width: 100%;
  margin: 0 auto;
}

.cart-page {
  padding: 24px;
}

.empty {
  margin: 16px 0;
  font-size: 14px;
  color: #6b7280;
}

.cart-list {
  list-style: none;
  padding: 0;
  margin: 0 0 16px;
}

.cart-item {
  background: #ffffff;
  border-radius: 12px;
  padding: 10px 12px;
  margin-bottom: 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.remove-btn {
  border: none;
  background: #e5e7eb;
  color: #111827;
  border-radius: 999px;
  padding: 6px 10px;
  font-size: 13px;
  cursor: pointer;
}

.checkout {
  background: #ffffff;
  padding: 16px 16px 14px;
  border-radius: 14px;
  box-shadow: 0 4px 10px rgba(15, 23, 42, 0.06);
}

.checkout input {
  width: 100%;
  padding: 8px 10px;
  margin: 6px 0;
  border-radius: 999px;
  border: 1px solid #d1d5db;
  font-size: 14px;
}

.checkout input.invalid {
  border-color: #dc2626;
}

.summary {
  display: flex;
  justify-content: space-between;
  margin-top: 8px;
  font-size: 14px;
}

.checkout-btn {
  width: 100%;
  margin-top: 12px;
  padding: 10px 0;
  border-radius: 999px;
  border: none;
  background: #4b5563;
  color: #ffffff;
  font-size: 14px;
  cursor: pointer;
}

.checkout-btn:disabled {
  background: #d1d5db;
  cursor: default;
}

.footer {
  margin-top: 30px;
  padding: 16px 0;
  background: #111827;
  color: #f9fafb;
  border-radius: 12px 12px 0 0;
}

.footer-content {
  display: flex;
  justify-content: center;
  gap: 40px;
  font-size: 0.95rem;
}

.footer-item {
  opacity: 0.85;
}

.footer-link {
  cursor: pointer;
  text-decoration: underline;
}

.footer-item:hover {
  opacity: 1;
}

.auth-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(15, 23, 42, 0.35);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 60;
}

.auth-modal {
  background: #ffffff;
  padding: 20px 20px 16px;
  border-radius: 16px;
  width: 90%;
  max-width: 360px;
  box-shadow: 0 12px 30px rgba(15, 23, 42, 0.25);
}

.auth-modal input {
  width: 100%;
  padding: 8px 10px;
  margin: 6px 0;
  border-radius: 999px;
  border: 1px solid #d1d5db;
}

.auth-actions {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
  margin-top: 10px;
}

.auth-primary {
  padding: 8px 12px;
  border-radius: 999px;
  border: none;
  background: #4b5563;
  color: #ffffff;
  font-size: 14px;
  cursor: pointer;
}

.auth-secondary {
  padding: 8px 12px;
  border-radius: 999px;
  border: 1px solid #d1d5db;
  background: #ffffff;
  font-size: 14px;
  cursor: pointer;
}
</style>
