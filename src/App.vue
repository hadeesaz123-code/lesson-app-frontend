<template>
  <div id="app">

    // header section
    <header class="header">

      // title area
      <div class="header-title">
        <h1>After School Activities</h1>
        <p>Discover activities for your child</p>
      </div>

      // top buttons and search
      <div class="header-actions">

        // search box
        <input
          v-model="searchQuery"
          placeholder="Search activities..."
          class="search-input"
        />

        // login button
        <button
          v-if="!currentUser"
          class="auth-btn"
          @click="openAuth('login')"
        >
          Login
        </button>

        // register button
        <button
          v-if="!currentUser"
          class="auth-btn"
          @click="openAuth('register')"
        >
          Register
        </button>

        // when logged in
        <div v-if="currentUser" class="auth-user">
          {{ currentUser.name }}
          <button class="auth-btn" @click="logout">Logout</button>
        </div>

        // cart button
        <button
          class="cart-toggle"
          :disabled="cart.length === 0"
          @click="toggleCart"
        >
          <span class="cart-label">
            {{ showCart ? "Back to lessons" : "View cart" }}
          </span>
          <span v-if="cart.length" class="cart-badge">
            {{ cart.length }}
          </span>
        </button>

      </div>
    </header>

    // main content
    <main class="layout">

      // lesson page
      <div v-if="!showCart">
        <LessonList
          :search="searchQuery"
          @add-to-cart="onAddToCart"
        />

        <footer class="footer">
          <div class="footer-content">
            <div class="footer-item footer-link" @click="goHome">Home</div>
            <div class="footer-item">Phone: 07123 456789</div>
            <div class="footer-item">Email: info@example.com</div>
          </div>
        </footer>
      </div>

      // cart page
      <div v-else class="cart-page">

        <h2>Your cart</h2>

        <div v-if="cart.length === 0" class="empty">
          Your cart is empty.
        </div>

        <ul v-else class="cart-list">
          <li
            v-for="(item, index) in cart"
            :key="item.id + '-' + index"
            class="cart-item"
          >
            <span>{{ item.subject }} — £{{ item.price }}</span>
            <button @click="removeFromCart(index)" class="remove-btn">
              Remove
            </button>
          </li>
        </ul>

        <div class="checkout">

          <h3>Checkout</h3>

          <input
            v-model="name"
            placeholder="Your name"
            :class="{ invalid: name && !validName }"
          />

          <input
            v-model="phone"
            placeholder="Phone number"
            :class="{ invalid: phone && !validPhone }"
          />

          <input
            v-model="email"
            placeholder="Email"
            :class="{ invalid: email && !validEmail }"
          />

          <div class="summary">
            <span>Total:</span>
            <strong>£{{ totalPrice.toFixed(2) }}</strong>
          </div>

          <button
            class="checkout-btn"
            :disabled="!canCheckout"
            @click="confirmOrder"
          >
            Confirm order
          </button>

        </div>
      </div>

    </main>

    // login/register box
    <div v-if="showAuthModal" class="auth-backdrop">
      <div class="auth-modal">

        <h2 v-if="authMode === 'login'">Login</h2>
        <h2 v-else>Create account</h2>

        <input v-model="authEmail" type="email" placeholder="Email" />
        <input v-model="authPassword" type="password" placeholder="Password" />

        <input
          v-if="authMode === 'register'"
          v-model="authName"
          placeholder="Full name"
        />

        <div class="auth-actions">
          <button class="auth-primary" @click="submitAuth">
            {{ authMode === "login" ? "Login" : "Create account" }}
          </button>
          <button class="auth-secondary" @click="closeAuth">Cancel</button>
        </div>

      </div>
    </div>

  </div>
</template>

<script>
// load LessonList
import LessonList from "./LessonList.vue";

// backend address
const API =
  window.location.hostname === "localhost"
    ? "http://localhost:3000"
    : "https://lesson-app-backend-7mte.onrender.com";

export default {

  // register LessonList
  components: { LessonList },

  // app data
  data() {
    return {
      searchQuery: "",
      showCart: false,
      cart: [],
      name: "",
      phone: "",
      email: "",
      showAuthModal: false,
      authMode: "login",
      authEmail: "",
      authPassword: "",
      authName: "",
      currentUser: null
    };
  },

  // run when app loads
  mounted() {
    const saved = localStorage.getItem("user");
    if (saved) {
      this.currentUser = JSON.parse(saved);
    }
  },

  // checks
  computed: {

    // letters only
    validName() {
      return /^[A-Za-z\s]+$/.test(this.name);
    },

    // numbers only
    validPhone() {
      return /^\d+$/.test(this.phone);
    },

    // email format
    validEmail() {
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(this.email);
    },

    // allow checkout
    canCheckout() {
      return (
        this.cart.length > 0 &&
        this.validName &&
        this.validPhone &&
        this.validEmail
      );
    },

    // total price
    totalPrice() {
      return this.cart.reduce((sum, item) => sum + item.price, 0);
    }
  },

  methods: {

    // add to cart
    onAddToCart(item) {
      this.cart.push(item);
    },

    // show cart
    toggleCart() {
      this.showCart = !this.showCart;
    },

    // remove item
    removeFromCart(index) {
      this.cart.splice(index, 1);
      if (this.cart.length === 0) this.showCart = false;
    },

    // home page
    goHome() {
      this.showCart = false;
      this.searchQuery = "";
      window.scrollTo(0, 0);
    },

    // open login
    openAuth(mode) {
      this.authMode = mode;
      this.showAuthModal = true;
      this.authEmail = "";
      this.authPassword = "";
      this.authName = "";
    },

    // close login
    closeAuth() {
      this.showAuthModal = false;
    },

    // login or register
    async submitAuth() {

      const endpoint =
        this.authMode === "login" ? "/login" : "/register";

      const payload = {
        email: this.authEmail,
        password: this.authPassword,
        name: this.authMode === "register" ? this.authName : undefined
      };

      try {
        const res = await fetch(`${API}${endpoint}`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(payload)
        });

        const data = await res.json();

        if (!res.ok) {
          alert(data.error || "Login failed");
          return;
        }

        this.currentUser = data.user || {
          name: payload.name,
          email: payload.email
        };

        localStorage.setItem("user", JSON.stringify(this.currentUser));

        alert(data.message);
        this.closeAuth();

      } catch (error) {
        alert("No connection");
      }
    },

    // logout
    logout() {
      this.currentUser = null;
      localStorage.removeItem("user");
      alert("Logged out");
    },

    // send order and update lessons
    async confirmOrder() {

      if (!this.canCheckout) return;

      const payload = {
        name: this.name,
        phone: this.phone,
        email: this.email,
        items: this.cart.map(item => ({
          lessonId: item.id,
          quantity: 1
        }))
      };

      await fetch(`${API}/orders`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
      });

      for (const item of this.cart) {

        const newSpaces = item.spaces - 1;

        await fetch(`${API}/lessons/${item.id}`, {
          method: "PUT",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ spaces: newSpaces })
        });
      }

      alert("Order done!");

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
